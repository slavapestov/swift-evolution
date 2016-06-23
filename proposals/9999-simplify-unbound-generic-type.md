# Simpler interpretation of a reference to a generic type with no arguments

* Proposal: [SE-9999](9999-simplify-unbound-generic-type.md)
* Author: [Slava Pestov](https://github.com/slavapestov)
* Status: **Awaiting review**
* Review manager: TBD

## Introduction

This proposal cleans up the semantics of a reference to a generic type
when no generic arguments are applied.

Swift-evolution thread: [Discussion thread topic for that proposal](http://news.gmane.org/gmane.comp.lang.swift.evolution)

## Motivation

Right now, we allow a generic type to be referenced with no generic
arguments applied in a handful of special cases. The two primary rules
here are the following:

* If the scope from which the reference is made is nested inside the
  definition of the type or an extension thereof, omitting generic
  arguments just means to implicitly apply the arguments from context.

  For example,

```swift
struct GenericBox<Contents> {
  let contents: Contents

  // Equivalent to: func clone() -> GenericBox<Contents>
  func clone() -> GenericBox {
    return GenericBox(contents: contents)
  }
}

extension GenericBox {
  func print() {
    // Equivalent to: let cloned: GenericBox<Contents>
    let cloned: GenericBox = clone()
    print(cloned.contents)
  }
}
```

* If the type is referenced from an unrelated scope, we attempt to
  infer the generic parameters.

  For example,

```swift
func makeABox() -> GenericBox<Int> {
  // Equivalent to: GenericBox<Int>(contents: 123)
  return GenericBox(contents: 123)
}
```

The problem appears when the user expects the second behavior, but
instead encounters the first. For example, the following does not
type check:

```swift
extension GenericBox {

  func transform<T>(f: Contents -> T) -> GenericBox<T> {
    // We resolve 'GenericBox' as 'GenericBox<Contents>', rather than
    // inferring the type parameter
    return GenericBox(contents: f(contents))
  }
}
```

## Proposed solution

The proposed solution is to remove the first rule altogether. If the
generic parameters cannot be inferred from context, they must be
specified explicitly with the usual `Type<Args...>` syntax.

## Detailed design

This really just involves removing an existing piece of logic from
the type resolver code.

## Impact on existing code

This will have a small impact on existing code that uses a pattern
similar to the above.

## Alternatives considered

### Status quo

We could keep the current behavior, but one can argue it is not very
useful, and adds a special case where one is not needed.

### More complex inference of generic parameters

We could attempt to unify the two rules for resolving a reference to
a generic type with no arguments, however this presents theoretical
difficulties with our constraint solver design. Even if it were
easy to implement, it would increase type checking type by creating
new possibilities to consider, with very little actual benefit.
