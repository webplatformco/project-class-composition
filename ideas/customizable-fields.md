# Customizable `[[ Fields ]]`

## Motivation

Public class fields made the common pattern of declaring instance properties more declarative.

However, while static class fields can be trivially introspected by looking at properties of the class constructor, there is no way to introspect instance fields from a class reference (without creating an instance), since they are not exposed on the class prototype.

This limits many metaprogramming use cases, as there is no way to figure out the shape of the class from the outside, without creating an instance.
Some such cases related to class partials are:
- Desugaring [class spread syntax](class-spread.md) (or implementing a similar feature in userland)
- Generating accessors to support [delegation](delegation.md) programmatically

Additionally, with **append mutations**, it could also address all the partials-related use cases around [constructor side effects](constructor-initializer.md) and facilitate even better DX for the delegation pattern (just one call to a utility method could both define the delegate property and add any necessary API surface).
This would also sidestep the issues around adding constructor side effects or initializers to built-ins, since built-in classes are not created via `ClassDefinitionEvaluation` and thus have no internal `[[ Fields ]]` slot.

Basically, read-only access addresses the use cases where a class is used as **input**, whereas append mutations address the use cases where a class is extended in-place.

## Proposal

### Core idea

A new known symbol that provides read-only or limited read-write access to (a subset of) a class's internal `[[ Fields ]]` slot.

### Design decisions

#### Public only or private too?

It may be simpler to expose **public fields only**, since that’s what most introspection use cases need, and this way every part of the code can access the same data structure.

#### Instance only or static too?

It could be argued that since static fields can already be introspected (though less cleanly) it may be worth only exposing **instance fields** for simplicity, as then the data structure could basically be a map of field names to initializers, with no other metadata.

However, including metadata is a more extensible design, in which case there is no benefit in excluding static fields.
This is also a cleaner transformation over the internal `[[ Fields ]]` value: just filter out private fields, done.

#### Mutable?

There is a spectrum between purely read-only access (which still addresses the use cases around introspection) and a completely mutable data structure, which allows adding fields, removing fields, or modifying initializers.

Since immutability tends to be relied on by VMs for performance optimizations, we should be very frugal in what types of mutations are allowed, and only support mutations that are well motivated by use cases.

Full mutation support does not seem to be necessary for partials use cases (though it would alleviate many of the decorator use cases).

While making the data structure fully mutable seems like it adds complexity without enough use cases to warrant it, the ability to **add** can be incredibly powerful, as it addresses most of the use cases for [instance initializers](constructor-initializer.md) while

One question in that case is how to expose an append-only _List_.
Possibly an array-like object with a `push` method?
Or would a completely exotic iterable with an `append` method be better?
If array-like, would it include mutation methods, and just throw or fail silently?
Essentially we need the opposite of `Object.preventExtensions()`: allow extensions, but freeze existing property assignments.
Basically an array with all its numerical properties defined as [[Writable]]: **false**, and [[Configurable]]: **false**.

#### Potential known symbol name

Depending on the design decisions above, the following are potential known symbol names:

- `Symbol.fields`
- `Symbol.instanceFields`
- `Symbol.publicFields`
- `Symbol.publicInstanceFields`

### Strawman

_Just for the sake of getting the discussion started._

The initial value of `Symbol.publicFields` is the well-known symbol `%Symbol.publicFields%`.

This property has the attributes { [[Writable]]: **false**, [[Enumerable]]: **false**, [[Configurable]]: **false** }.

It is an accessor on the class constructor function object, so it’s computed from `[[ Fields ]]` and can stay consistent with decorators or other transforms that might reorder/augment during definition time.

The getter returns an append-only _List_ of _Record_ objects, each with the following properties:
- `name`: The name of the field, same as [[Name]] on [ClassFieldDefinition](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-classfielddefinition-record-specification-type). (`string` or `Symbol`)
- `initializer`: The initializer of the field, same as [[Initializer]] on [ClassFieldDefinition](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-classfielddefinition-record-specification-type). (`Function`)
- `isStatic`: Whether the field is static. (`boolean`)

The list is exposed using an array-like object whose numerical properties are non-writable and non-configurable.
Array methods are supported, but any attempt to modify the list will throw a `TypeError` consistent with any other attempt to modify a non-writable and non-configurable property.
