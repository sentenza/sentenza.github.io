---
layout: post
title: "About Type Classes"
description: "What is exactly a Type Class in Scala"
category: scala
tags: [Scala]
---

We've already gone through the introduction of what is actually a Monad in [a previous article][monadic-1] and we discovered that all the classes that share the same functionality form a sort of Category. Now I want to show you how to apply those functionalities to a wide range of other classes without the need of using OOP style or either touching pre-existent code. How can I do that? It's simple, using **Type Classes**, so let's dive in!

## Introducing the Type Class concept

Now we're going to take advantage of a technique that was originally introduced in Haskell: **Type Class** definition. The idea is to define a _group of types_ (_a class of types_) that will share a common contract-defined behaviour. At first, one could thought to do that by extending a common trait, using OOP and ["coding to interfaces"][interface-based programming].

Instead a type class do not use sub-typing. _Dotty_ (Scala 3) [defines types classes][dotty-type-class] as follows:

> A type class is an abstract, parameterized type that lets you add new behavior to any closed data type without using sub-typing. This can be useful in multiple use-cases, for example:
>
> * expressing how a type you don't own (from the standard or 3rd-party library) conforms to such behavior
> * expressing such a behavior for multiple types without involving sub-typing relationships (one `extends` another) between those types (see: [ad hoc polymorphism](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism) for instance)

### Type Classes in Scala 2

In Scala 2 a Type Class is defined as a three-step pattern:

1. a polymorphic trait (e.g. `trait MyTypeClass[A]`) with one or more abstract methods;
2. different applications of the `implicit` keyword to provide the so-called **type class syntax**;
3. the instances of the type class, to provide implementations of the type class for specific types that we're willing to "extend".

Let's use the most simple [Category][category_theory] - SemiGroup - and an [example extracted from Heiko Seeberger's blog][seeberger_type_classes]:

```scala
// 1. Type class as a polymorphic trait
trait SemiGroup[A] {
  def combine(a1: A, a2: A): A
}

// 2. The type class syntax is provided by extension methods
//    with an implicit parameter for the type class
final implicit class SemiGroupSyntax[A](val lhs: A) extends AnyVal {
  final def |+| (rhs: A)(implicit sga: SemiGroup[A]): A =
    lhs.combine(rhs)

  def combine(rhs: A)(implicit sga: SemiGroup[A]): A =
    sga.combine(lhs, rhs)
}

// 3. The type class instance is an implicit value
//    implementing the type class
final implicit val intSemiGroup: SemiGroup[Int] =
  _ + _
```

As you have seen so far **working with type classes in Scala means working with implicit values and implicit parameters**. This is something that we need to do for the compiler to be able to solve the dependencies for us and to give the end user an easy to use "extension methods toolkit" out of the box, that looks like the sub-typing looking at the resulting syntax.

As a simple experiment we can see what would have happened if the implicit were not part of the equation:

```scala
object SemiGroup {
  val intSemiGroup = 
    new SemiGroup[Int] {
      def combine(lhs: Int, rhs: Int) = lhs + rhs
    }
}
```

Easy to read but cumbersome when it comes to its actual usage. At first we define a companion object for SemiGroup trait that holds an implementation (an **instance**) for the `Int` type. So far so good, but to use our new instance of SemiGroup we have to call `intSemiGroup` directly:

```scala
println(intSemiGroup.combine(5, 8)) // => 13
```

Could we improve that and then use `combine()` without declaring `intSemiGroup` explicitly? Well, as you might expect we can do that "implicitly":

```scala
object SemiGroup {
  // Passing the generic SemiGroup to combine()
  def combine[A](lhs: A, rhs: A)(implicit sg: SemiGroup[A]) = sg.combine(lhs, rhs)

  // This represents the INSTANCE of SemiGroup for Int
  // Redefining the instance using implicit
  implicit val intSemiGroup: SemiGroup[Int] =
    new SemiGroup[Int] {
      def combine(lhs: Int, rhs: Int): Int = lhs + rhs
    }
}
```

Now the compiler should be able to find the `intSemiGroup` instance for us and pass that instance to our combine method whenever we _combine_ two integers:

```scala
println(combine(8, 13)) // => 21
```

We have a trait that describes the functionality and implementations (**instances**) for each type, in our case `Int`, that we care about. On top of that we've also defined a function which applies an implicit instance function to the given parameters.

We can improve the code even more to make it look like production code. Looking at the signature of the combine function `combine[A](lhs: A, rhs: A)(implicit sg: SemiGroup[A])` we could take advantage of `implicitly` to "summon" the generic type `SemiGroup[A]`:

```scala
def combine[A: SemiGroup](lhs: A, rhs: A) = implicitly[SemiGroup[A]].combine(lhs, rhs)
```

Note that `A: SemiGroup` is a **context bound syntax** ([info][scala-context-bound]), which is a syntactic sugar in Scala, mainly introduced to support type classes. Basically, it does the rewrite we have done above (without the use of implicitly).

Another useful trick that is often used by convention when defining type classes is to introduce an `apply` function having only an implicit parameter, instead of using `implicitly`:

```scala
def apply[A](implicit sg: SemiGroup[A]): SemiGroup[A] = sg

// Using implicitly
def apply[A]: SemiGroup[A] = implicitly[SemiGroup[A]]
```

Please note that I used the term "summon" on purpose. As a matter of fact, we can introduce a new utility trait named `Summoner` to do that for us (see the Trait defined below). Further reading about that can be also found [here][summoner].

```scala
trait Summoner[T[_[_]]] {
  def apply[A[_]: T]: T[A] = implicitly[T[A]]
}
```

We can improve our type class even further adding the "inner ability" of calling the `combine()` function as if it were a method on the given object - by using a simple `.combine` notation. By convention it's commonly called a _**Ops class**_.

```scala
implicit class SemiGroupOps[A: SemiGroup](lhs: A) {
  def combine(rhs: A) = SemiGroup[A].combine(lhs, rhs)
}
```

so we could call the combine function on an object that has an instance of our type class in scope:

```scala
val x: Int = 5
val y: Int = 9
x combine y // 14
```

To avoid a runtime overhead it's possible to transform our `SemiGroupOps` into a value class and move the type class constraint to the combine function, like the following:

```scala
implicit class SemiGroupOps[A](val lhs: A) extends AnyVal {
  def combine(rhs: A)(implicit sg: SemiGroup[A]) = sg.combine(lhs, rhs)
}
```

So down the line our SemiGroup type class should look like this:

```scala
object SemiGroup {
  def apply[A]: SemiGroup[A] = implicitly[SemiGroup[A]]

  def combine[A: SemiGroup](lhs: A, rhs: A) = SemiGroup[A].combine(lhs, rhs)

  implicit class SemiGroupOps[A: SemiGroup](val lhs: A) {
    def combine(rhs: A) = SemiGroup[A].combine(lhs, rhs)
  }

  implicit val intSemiGroup: SemiGroup[Int] =
    new SemiGroup[Int] {
      def combine(lhs: Int, rhs: Int): Int = lhs + rhs
    }

  // A shorter version could be
  implicit val longSemiGroup: SemiGroup[Long] = 
    (lhs: Long, rhs: Long) => lhs + rhs
}
```

To avoid the implicit clash given by several imports it's a good idea to move the combine function into an Ops object to allow users of this type class to redefine the default instance behaviour.

```scala
object SemiGroup {
  def apply[A]: SemiGroup[A] = implicitly[SemiGroup[A]]

  object ops {
    def combine[A: SemiGroup](lhs: A, rhs: A: A = lhs + rhs

    implicit class SemiGroupOps[A: SemiGroup](val lhs: A) {
      def combine(rhs: A) = SemiGroup[A].combine(lhs, rhs)
    }
  }

// intSemiGroup...
}
```

Usage doesn't really change but now we may import only

```scala
import SemiGroup
import SemiGroup.ops._
```

As a final note about Type Classes in Scala I would suggest the reading of [Type Classes in Cats](https://typelevel.org/cats/typeclasses.html) reporting a diagram of all the type classes introduced by the Cats library. Spoiler: it's a lot of type classes. :)

### Further readings and resources

* [Type classes in Scala](https://scalac.io/typeclasses-in-scala/)
* [Scala with Cats - The Type Class](https://www.scalawithcats.com/dist/scala-with-cats.html#the-type-class)
* [Type class - Dotty][dotty-type-class]
* [Type classes in Dotty](https://heikoseeberger.rocks/2019/12/10/2019-12-10-dotty-3/)
* [Scala Context Bound syntax][scala-context-bound]
* [Type Classes in Scala 3 Dotty][seeberger_type_classes]
* [Implementing Type Classes in Scala 3](https://dotty.epfl.ch/docs/reference/contextual/type-classes.html)

[interface-based programming]: https://en.wikipedia.org/wiki/Interface-based_programming
[dotty-type-class]: https://dotty.epfl.ch/docs/reference/contextual/type-classes.html
[monadic-1]: /2020/12/10/scala-monadic-journey-1/
[category_theory]: /docs/category_theory/
[scala-context-bound]: https://docs.scala-lang.org/tutorials/FAQ/context-bounds.html
[summoner]: /docs/category_theory/#summoner
[seeberger_type_classes]: https://heikoseeberger.rocks/2019/12/10/2019-12-10-dotty-3/ 
