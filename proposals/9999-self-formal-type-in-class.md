# Consistent formal type for 'self' in class methods

* Proposal: [SE-9999](9999-self-formal-type-in-class.md)
* Author: [Slava Pestov](https://github.com/slavapestov)
* Status: **Awaiting review**
* Review manager: TBD

## Introduction

This proposal makes the `self` value behave consistently whether or
not it is used from a method with a `Self` return type.

Swift-evolution thread: [Discussion thread topic for that proposal](http://news.gmane.org/gmane.comp.lang.swift.evolution)

## Motivation

Right now, we exhibit inconsistent behavior when `self` is used as
an argument to a generic function, violating the principle of least
surprise.

Consider the following code:

```swift
class Base {
  @discardableResult
  func methodWithDynamicSelf() -> Self {
    doSomething(self)
    return self
  }

  func methodWithoutDynamicSelf() {
    doSomething(self)
  }
}

class Derived : Base {}

func doSomething<T>(_ t: T) {
  print(T.self)
}

Base().methodWithDynamicSelf()
Base().methodWithoutDynamicSelf()

Derived().methodWithDynamicSelf()
Derived().methodWithoutDynamicSelf()
```

Currently, it prints the following output:

```swift
Base
Base
Derived
Base
```

Note that there's no inconsistency when the method is called on the
base class. When called on the derived class however, we see that
in a method with a dynamic `Self` return type, the type of `self` is
`Derived`, whereas in a method with any other return type, the type
of `self` is `Base`.

## Proposed solution

The proposal is to change the type of `self` to always be `Self`, which
can be thought of as a special generic type parameter bound to the
dynamic type of the instance.

With this proposal, the above code will instead produce the following:

```swift
Base
Base
Derived
Derived
```

Here, the type of `self` would always be `Derived` when called on an
instance of the derived class.

Of course a more useful program could instead do something with the
type parameter `T`, such as constraining it to a protocol or a class
with a required initializer, and then using the type to construct
a new instance of the class.

This also dovetails nicely with [SE-0068](0068-universal-self.md).

Finally, it opens the door to generalizing dynamic `Self`, allowing
it to appear in covariant position within parameter types:

```swift
class ArtClass {
  func paint(withBrush: (Self) -> ()) { ... }
}
```

This would allow a class to conform to a protocol with a requirement
written like the following, something that is currently not possible
at all:

```swift
protocol OddProtocol {
  func weaken<X, Y>((Self) -> (X) -> Y) -> (X) -> Y
}
```

## Detailed design

There's really not much more to say here. The code for typing `self`
with a dynamic `Self` is in place already, however enabling this change
might expose some new bugs we have not yet encountered, because
currently, methods with dynamic `Self` return type are relatively rare.

## Impact on existing code

This will have a small impact on existing code that uses a pattern
similar to the above.

## Alternatives considered

One alternative is to simply do nothing, but this makes the language
less consistent than it could be.
