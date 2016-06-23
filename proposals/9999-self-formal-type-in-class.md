# Feature name

* Proposal: [SE-9999](9999-self-formal-type-in-class.md)
* Author: [Slava Pestov](https://github.com/slavapestov)
* Status: **Awaiting review**
* Review manager: TBD

## Introduction

This proposal makes the 'self' value behave consistently whether or
not it is used from a method with a 'Self' return type.

Swift-evolution thread: [Discussion thread topic for that proposal](http://news.gmane.org/gmane.comp.lang.swift.evolution)

## Motivation

Right now, we exhibit inconsistent behavior when 'self' is used as
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

With this proposal, the above code will instead produce the following:

```swift
Base
Base
Derived
Derived
```

Of course a more useful program could instead do something with the
type parameter 'T', such as constraining it to a protocol or a class
with a required initializer, and then using the type to construct
a new instance of the class.

## Proposed solution

The proposal is to change the type of 'self' to always be 'Self', and
not the static type of the class.

## Detailed design

There's really not much more to say here. The code for dynamic 'Self'
is mostly in place already, however enabling this change might expose
some new bugs we have not yet encountered, because currently, methods
with dynamic 'Self' return type are relatively rare.

## Impact on existing code

This will have a small impact on existing code that uses a pattern
similar to the above.

## Alternatives considered

One alternative is to simply do nothing, but this makes the language
less consistent than it could be.
