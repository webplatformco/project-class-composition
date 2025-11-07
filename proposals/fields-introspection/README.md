# Introspection of public [[Fields]]

## Status

Champion(s): Lea Verou

Author(s): Lea Verou

Stage: 0

## Motivation

Public class fields made the common pattern of declaring instance properties more declarative.

However, while static class fields can be trivially introspected by looking at properties of the class constructor, there is no way to introspect instance fields from a class reference without creating an instance (which is not always practical, or even allowed), since they are not exposed on the class [[Prototype]].

This limits many metaprogramming use cases, as there is no way to figure out the shape of the class from the outside, without creating an instance.

## Use cases

For example, this includes all the use cases for [class spread syntax](../class-spread) and other similar features.

> TBD: expand on use cases

## Proposal

A new known symbol that provides read-only access to a class's internal `[[ Fields ]]` slot, or a subset.

### Design decisions

#### Public only or private too?

While at first it seems like exposing private fields would violate encapsulation, it does not reveal anything that is not already revealed by simply accessing the class's [[SourceText]].

However, if exposing public fields only is easier to implement, that would be a valid trade-off, since most use cases are about introspecting public API surface.

#### Instance only or static too?

It could be argued that since static fields can already be introspected (though less cleanly) it may be worth only exposing **instance fields** for simplicity, as then the data structure could basically be a map of field names to initializers, with no other metadata.

However, including metadata is a more extensible design, in which case there is no benefit in excluding static fields.
This is also a cleaner transformation over the internal `[[ Fields ]]` list: just filter out private fields, done.

#### Data structure should support future mutability

While it seems prudent to only expose a read-only data structure, it should be designed to support (limited) future mutability, as there are many use cases that could benefit from even append-only mutations.

For example, a frozen array would fulfill that requirement, as it can later evolve to a regular array (with some properties non-writable and non-configurable) without breaking existing code.

#### Potential known symbol name

Depending on the design decisions above, the following are potential known symbol names:

- `Symbol.fields`
- `Symbol.instanceFields`
- `Symbol.publicFields`

### Strawman

_Just for the sake of getting the discussion started._

The initial value of `Symbol.publicFields` is the well-known symbol `%Symbol.publicFields%`.

This property has the attributes { [[Writable]]: **false**, [[Enumerable]]: **false**, [[Configurable]]: **false** }.

It is an accessor on the class constructor function object, so it’s computed from `[[ Fields ]]` and can stay consistent with decorators or other transforms that might reorder/augment during definition time.

The getter returns an append-only _List_ of _Record_ objects, each with the following properties:
- `name`: The name of the field, same as [[Name]] on [ClassFieldDefinition](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-classfielddefinition-record-specification-type). (`string` or `Symbol`)
- `initializer`: The initializer of the field, same as [[Initializer]] on [ClassFieldDefinition](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-classfielddefinition-record-specification-type). (`Function`)
- `isStatic`: Whether the field is static. (`boolean`)

The list is exposed using a frozen array.
