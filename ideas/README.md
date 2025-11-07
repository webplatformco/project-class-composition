# Class Partials Ideas

This directory contains various ideas around features that could pave the way towards better userland and native class partials in some way.
More mature ideas may graduate to [proposals](./proposals).

The more mature proposals around that are:

- [Customizable public `[[ Fields ]]`](ideas/customizable-fields.md): Read-only or possibly read plus append-only access to the public fields of a class's internal `[[ Fields ]]` slot through a new known symbol property.
- [Class spread](ideas/class-spread.md): Spread syntax for class definitions, to facilitate behavior sharing without affecting the inheritance chain.

These are earlier in their incubation:

- [`new Function(fnObject)`](ideas/function-constructor.md): Modernize the Function constructor to take function objects and clone them, rather than relying on serialization.
- [Instance initializers / Customizable `[[Construct]]`](ideas/constructor-initializer.md): Allow for instance initializers that run whenever a prototype is assigned and/or after the constructor has run.
- [Customizable `[[Call]]`](ideas/customizable-call.md): Allow for the `[[Call]]` internal method to be customized through a new known symbol property.
- [Mutable Functions](ideas/mutable-functions.md): A new function type: `MutableFunction` subclass of `Function` that allows for side effects that can be added and removed after the function has been created.

And these are not even drafts yet:

- [Forwarding](ideas/forwarding.md): A way to forward properties and methods to a delegate object, a bit like a cross between a Proxy and accessors.
- [Delegation](ideas/delegation.md): Do we need a declarative way to make the delegation pattern easier?


