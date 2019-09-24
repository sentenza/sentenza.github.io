---
layout: post
title: "Scala & Monoids: a step into the Category Theory"
description: "Dive into the Category Theory starting with Monoid in Scala"
category: scala
tags: [category theory, monoids]
---

In this post we're going to see what is a *Category* and in particular a *Monoid*, and to dive into this subject we'll take advantage of the Scala programming language. So let's start from the beginning and let's try to keep the things simple and clear, because as we'll see in a moment, the *Monoid* is a very simple structure defined **only by its algebra**.

**In [category theory](http://en.wikipedia.org/wiki/Category_theory), a [monoid](http://en.wikipedia.org/wiki/Monoid_(category_theory)) is a [category](http://en.wikipedia.org/wiki/Category_(mathematics)) with one object.**

Taken alone, the previous statement says very little and doesn't buy us anything. The reason of this confusion is that the term *monoid* comes from mathematics and we need to analyse what is a category to fully understand its meaning. We'll do that, but for now let's define a monoid in terms of its laws.

### A Category? What is this sorcery?

A category is a purely algebraic structure consisting of "objects" and "arrows" between them, much like a directed graph with nodes and edges between them. A category will have objects like `X`, `Y`, `Z`,etc. and arrows between objects. Importantly, arrows **compose**. Given an arrow _f_ from `X` to `Y`, and another arrow _g_ from `Y` to `Z`, their composition is an arrow from `X` to `Z`. There is also an identity arrow from every object to itself.

![Category diagram](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ef/Commutative_diagram_for_morphism.svg/400px-Commutative_diagram_for_morphism.svg.png)


## Let's define a monoid
A monoid consists of:

- some type `T`;
- an **associative binary operation**, `op`, that takes two values of type `T` and combines them into one: `op(op(x,y), z) == op(x, op(y,z))` for any choice of `x: T`, `y: T`, `z: A`;
- A value, `zero: T`, that is an **identity for that operation**: `op(x, zero) == x` and `op(zero, x) == x` for any `x: T`.

```scala
trait Monoid[T] {
  def op(x: T, y: T): T
  def zero: T
}
```
So the thing is that monoids are everywhere, and we use them continously without even noticing it. When we work with lists or when we accumulate results of a loop we might want to express these operations in terms of monoids. Essentially, we can try to express common operations using their algebra to factor common traits. 

Above all, monoids are useful in two important ways:

- they make it easy doing parallel computations, because we'll be able to split our problem in smaller pieces
- they can **be composed** to form complex computations out of simple pieces.

### Monoids, Monoids, Monoids
<iframe width="560" height="315" src="https://www.youtube.com/embed/DJyhWAwmGqE" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### OK, fine. Give me some code

To use the `Monoid[T]` trait we need some instances, so let's define them and you'll see in a minute how easy actually is to factor the common algebraic behaviours of common types.

```scala
val floatAddition: Monoid[Float] = new Monoid[Float] {
    def op(x: Float, y: Float): Float = x + y
    val zero: Float = 0
}

val floatMultiplication: Monoid[Float] = new Monoid[Float] {
    def op(x: Float, y: Float): Float = x * y
    val zero: Float = 1.0
}

val booleanOr: Monoid[Boolean] = new Monoid[Boolean] {
    def op(a: Boolean, b: Boolean) = a || b
    val zero: Boolean = false
}

val booleanAnd: Monoid[Boolean] = new Monoid[Boolean] {
    def op(a: Boolean, b: Boolean) = a && b
    val zero: Boolean = true
}

val optionMonoid[U]: Monoid[Option[U]] = new Monoid[U] {
    def op(op1: Option[U], op2: Option[U]): Option[U] = op1 orElse op2
    val zero: Option[U] = None
}
```

Look for a moment at the last one, the Option Monoid. What happens if we pass two `Some` to the `op`? Only the first option
will be evaluated, so we might argue that this implementation is not the same as writing `def op(op2: Option[T], op1: Option[T])`. The truth is that we can compose the arguments of `op` in the opposite order; the monoid laws will remain intact but we will end up with a doppelgänger. 

![dopperganger](https://proxy.duckduckgo.com/iu/?u=http%3A%2F%2Fi735.photobucket.com%2Falbums%2Fww358%2FBlackOut0189%2FTwin%2520Peaks%2Fvlcsnap-00004.jpg&f=1)

Every monoid has a **_dual_** where the `op` combines things in the opposite order. Monoids like
 `floatAddition` are equivalent to their duals because their `op` is commutative as well as associative.

```scala
def dual[T](m: Monoid[T]): Monoid[T] = new Monoid[T] {
  def op(x: T, y: T): T = m.op(y, x)
  val zero = m.zero
}

// Now we can have both OptionMonoids on hand:
def firstOptionMonoid[U]: Monoid[Option[U]] = optionMonoid[U]
def dualOptionMonoid[U]: Monoid[Option[U]] = dual(firstOptionMonoid)
```

### Resources

- [Haskell/Monoids](https://en.wikibooks.org/wiki/Haskell/Monoids)
- [Monads are Monoids in the category of endofunctions](https://stackoverflow.com/q/9745994/1977778)
- [A companion boolet to _Functional Programming in Scala_](http://blog.higher-order.com/assets/fpiscompanion.pdf)
