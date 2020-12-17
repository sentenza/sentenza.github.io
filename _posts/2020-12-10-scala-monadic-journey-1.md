---
layout: post
title: "A monadic journey in Scala - Part 1"
description: "Your first Monad - Writer Monad"
category: scala
tags: [Scala]
---


And here we are, facing the deaf stone in front of us. We want to go functional and to do that we have to face the most important concept that we must embrace: **Monads**. In this post I'm going to show what is a Monad in Scala **in a very pragmatic way**, so you won't have to worry about strange definitions taken from the category theory or things like that. The goal here is to understand and use monads in Scala.

---

### Diving in: the scary "M" word

The first rule of the _monad club_ is don't google for _"monad"_ . . . If you try to do that a Google search brings up the following definitions:

- A philosophical concept: a single, indivisible and ultimately simple entity, such as an atom or a person
- From biology: a flagellate protozoan
- A mystical idea: the totality of all being
- For no discernible reason: a type of burrito

I took this list from [this presentation][scala-state-monads] and don't say I didn't warn you about that. Anyway, if you try to look for the exact definition of the term Monad in Category Theory you'll end up with something like that:

> A _Monad_ is a _Functor_ equipped with a pair of "natural transformations" satisfying the laws of associativity and identity.

It doesn't help too much, right? So, let's get back to what we stated at the very beginning: we want to introduce this concept in a very pragmatic way, no FP jargon but just what's worth to know in order to understand why every FP developer talks about Monads all day long.

### Back to the roots: Functional Programming

In order to really understand what is a monad we have to go one step back, mainly because it's a lot easier to uncover the actual meaning behind monads in Scala if we ask ourselves _"why do we need monads so badly?"_. And the answer is: because we need them as **a _super-glue_ for getting pure functions to work together**.

It all starts with Functional Programming. In his book ["Functional Programming, Simplified"][fp-simplified], Alvin Alexander defines _Functional Programming_ (FP) with just two statements:

1. FP is about writing software applications **using only pure functions**.
2. When writing FP code you should use only immutable values.

As a consequence we need to write pure functions if we want to use FP and pure functions _tell no lies_:

- the output of a pure function depends only on its input parameters and it has no _side effects_ (e.g. _I/O_ operations);
- pure functions _never_ throw exceptions (side effects);
- pure functions signatures are a contract with their consumers;
- Scala/FP developers use `Option` or `Either` rather than exceptions;
- Scala/FP developers use `Option` rather than `null` values;
- Scala constructs like `match` and `for` work great with `Option`.

#### A quick detour: the benefits of functional programming

> In computer science, functional programming is a programming paradigm where programs are constructed by applying and composing functions. It is a declarative programming paradigm in which function definitions are trees of expressions that each return a value, rather than a sequence of imperative statements which change the state of the program.
>
> -- [Wikipedia: Functional Programming](https://en.wikipedia.org/wiki/Functional_programming)

["Functional Programming, Simplified"][fp-simplified] gives the following list of benefits provided by the application of the functional programming principles:

- Pure functions are easier to reason about
- Testing is easier, and pure functions lend themselves well to techniques like [property-based testing][property-based-testing-in-scala]
- Debugging is easier
- Programs are written at a higher lever, and are therefore easier to comprehend
- Function signatures are more meaningful
- Parallel/Concurrent programming is easier
- Being able to treat functions as values and use anonymous functions makes code more concise, and still readable

### Writing FP programs in Scala leads to _for expressions_

Now we are ready to write some Scala code to prove how all those concepts are glued together. Let's start with the definition of a pure function, because FP is all about pure functions and nothing else essentially. We want to write a function that will return an amount of points based on a string that represents the final result of a match. Then we want to build a player score based on those results.

Let's start defining a [Sum ADT][scala-adt] to describe the possible results of a match and the corresponding points that will be assigned to a player.

```scala
sealed trait MatchResult {
  val points: Int
}

object MatchResult {
  case object Win  extends MatchResult { override val points = 3 }
  case object Tie  extends MatchResult { override val points = 1 }
  case object Loss extends MatchResult { override val points = 0 }
}
```

As a reference I'm going to introduce a new function that is clearly not pure, because it will throw an exception if something goes wrong (`f: String -> Int`)

```scala
def getMatchPointsNotPure(result: String): Int = result match {
  case "win"  => MatchResult.Win.points
  case "loss" => MatchResult.Loss.points
  case "tie"  => MatchResult.Tie.points
  case _      => throw new Exception("You passed an invalid result!") 
}
```

The previous function is not pure, even though the signature doesn't tell it clearly. To demonstrate it we suppose to have a list of strings that represents the player's results, but somehow someone screwed it up and the last result has a typo:

```scala
val results = List("win", "win", "loss", "tie", "win", "looss")
// Executing the program using a for loop
  try {
    results.foreach(r => println(s"Not pure result: $r => ${getMatchPointsNotPure(r)}"))
  } catch {
    case e: Exception => println(s"ERROR: ${e.getMessage()}")
  }
// We get the following results back
// # => Not pure result: win => 3
// # => Not pure result: win => 3
// # => Not pure result: loss => 0
// # => Not pure result: tie => 1
// # => Not pure result: win => 3
// # => ERROR: You passed an invalid result!
```

It's clear that if a value doesn't correspond to a valid string we'll end up with an Exception and not an Int. Let's rethink our function taking into account the possibility to have an _optional output_. We know that the result of `getMatchPointsNotPure()` could be either an integer or an "exceptional" value. Rephrasing the last sentence we can say that the function could return either _some_ value or _none_. To describe this situation we can use the Scala Option type. 

Option is a type that can be thought either as a _wrapper_ or a _box_. The new function will be pure and the results will be _lifted to an `Option[_]`_, therefore the resulting signature is `(result: String) => Option[Int]`:

```scala
def getMatchPoints(result: String): Option[Int] = result match {
  case "win" => Some(MatchResult.Win.points)
  case "loss" => Some(MatchResult.Loss.points)
  case "tie" => Some(MatchResult.Tie.points)
  case _ => None
}
``` 

Using the new pure function the generated points will be:

```
Result: win => Some(3)
Result: win => Some(3)
Result: loss => Some(0)
Result: tie => Some(1)
Result: win => Some(3)
Result: looss => None
```

All right, we have our pure function, but what now? How can we use those `Some(_)` boxed/lifted optional values to compute a score? It's tricky, isn't it? You may know that there are several ways to "unbox" those values and sum them. Having a list we could _fold_ that sequence, but let's use a for comprehension to do that. 

```scala
  val vect = results.toVector
  val score = for {
    a <- getMatchPoints(vect(0))
    b <- getMatchPoints(vect(1))
    c <- getMatchPoints(vect(2))
    d <- getMatchPoints(vect(3))
    e <- getMatchPoints(vect(4))
  } yield a + b + c + d + e  

  println(score.getOrElse(0))

  // # => 10
```

The for comprehension can be rewritten using Option's `flatMap` and `map` functions, but the for comprehension is a lot easier to read and reason about. For example let's just take the first two results, using `flatMap` and `map` we can sum the inner values as follows:

```scala
getMatchPoints(vect(0)).flatMap { a => 
    a.map(b => a + b)
} // Some(6)
```

This is how for expressions are converted by the Scala compiler. It's important to keep in mind that **any class that implements `map` and `flatMap` can be used in a for expression in the same way we used `Option` in the previous example**.

### Monads don't have to be scary

Even though this is not 100% true we could say that **any Scala class that implements `map` and `flatMap` is a _Monad_ and it can be used in a for expression**. What is more, Scala for expression is a sort of superglue for getting pure functions to work together.

> The only thing monads are relevant for, from a language perspective, is the ability of being used in a for-comprehension.
>
> `flatMap` is a basic requirement, and you can **optionally** provide `map`, `withFilter` and `foreach`.
>
> However, there's no such thing as strict conformance to a `Monad` typeclass, like in Haskell.
>
> -- [Gabriele Petronella - What exactly makes Option a monad in Scala?](https://stackoverflow.com/a/25361305/1977778)

I said that this was not 100% true because, from a theoretically point of view, a Monad should also:

- support a `unit` operation to create a monad out of a bare value (e.g. `apply()`)
- respect the [monadic laws][monadic-laws]

### Writing your first Monad in Scala

Let's create our first Monad that for now it's just a simple and generic _"Wrapper"_ for basic types like Int or String.

```scala
class Wrapper[T] private (value: T) {
  def map[R](f: T => R): Wrapper[R] = {
    val newValue = f(value)
    new Wrapper(newValue)
  }

  def flatMap[R](f: T => Wrapper[R]): Wrapper[R] = {
    val newValue = f(value)
    newValue
  }

  override def toString = value.toString
}

object Wrapper {
  // Factory Method
  def apply[T](value: T): Wrapper[T] = new Wrapper(value)
}
```

We used a Factory Method to let the end user a single point of access for creating a new Wrapper. What's more, when we want to "wrap" a new value into our new Wrapper Monad we're going to **lift** a value and then we actually support a `unit` operation to create a monad out of a bare value, which satisfy one of the conditions that we reported above.

```scala
val wrapped = Wrapped(500) // we lift our value 500 into a Wrapper[Int]
```

### Now a real use case: the Writer Monad

The real use case that we want to map here is writing logs while performing some operations, e.g. applying functions to values using Monads. So, we might want to start from our Wrapper class and then perform some operations:

```scala
def op1(x: Int): Wrapper[Int] = Wrapper(x + 1)
def op2(x: Int): Wrapper[Int] = Wrapper(x + 2)
def op3(x: Int): Wrapper[Int] = Wrapper(x + 3)

for {
  a <- op1(10) // Wrapper(11)
  b <- op2(20) // Wrapper(22)
  c <- op3(30) // Wrapper(33)
} yield a + b + c // Wrapper(66)
```

Now, you can imagine yourself adding some logs, to debug your functions if something goes wrong. But we cannot do any I/O operation if we want to keep or opX functions pure. So we cannot write something like:

```scala
def op1(x: Int): Wrapper[Int] = {
  val result = x + 1  
  logger.info(s"the wrapped result is Wrapper($result)") // e.g. println(...)
  Wrapper(result)
}
```

We need to carry across the logs with our values, so we might want to create a new function to return values with the logs:

```scala
def opl(x: Int): (Int, String) // but the tuple (Int, String) is not a Monad
```

We cannot use our Wrapper class here, because it carries just one type hence writing `Wrapper[Int, String]` will raise an error. Moreover, we need keep writing logs while performing operations. Why not thinking about something like `SomeWrapper[Int, List[String]]` then?

```scala
case class Writer[T](value: T, log: List[String]) {
  def map[R](f: T => R): Writer[R] = {
    val nextValue = f(value)
    Writer(nextValue, this.log)
  }

  def flatMap[R](f: T => Writer[R]): Writer[R] = {
    val nextValue: Writer[R] = f(value)
    Writer(nextValue.value, this.log ::: nextValue.log)
  }
}
```

And now our `opX` functions can be rewritten using our new Writer Monad:

```scala
object WriterTest extends App {
  def writerOp1(x: Int): Writer[Int] = 
    Writer(x + 1, List(s"$x + 1 = ${x + 1}"))
  def writerOp2(x: Int): Writer[Int] = 
    Writer(x * 2, List(s"$x + 2 = ${x * 2}"))
  def writerOp3(x: Int): Writer[Int] = 
    Writer(x * 3, List(s"$x + 3 = ${x * 3}"))

  val result = for {
    a <- writerOp1(10)
    b <- writerOp2(a)
    c <- writerOp3(b)
  } yield c

  result.log.foreach(l => println(s"LOG: $l"))
  println(s"Output: ${result.value}")
}
```

The final result is:

```scala
LOG: 10 + 1 = 11
LOG: 11 + 2 = 22
LOG: 22 + 3 = 66
Output: 66
```

### Further readings and resources

- [State Monads in Scala][scala-state-monads]
- [Functional Programming, Simplified][fp-simplified] by A. Alexander
- [A practical introduction to Functional Programming][cook-functional-programming]
- [Algebraic Data Types in Scala][scala-adt]

[scala-state-monads]: https://youtu.be/HZUi5XcsaNE
[fp-simplified]: https://www.goodreads.com/book/show/36453393-functional-programming-simplified
[cook-functional-programming]: https://maryrosecook.com/blog/post/a-practical-introduction-to-functional-programming
[property-based-testing-in-scala]: https://medium.com/analytics-vidhya/property-based-testing-scalatest-scalacheck-52261a2b5c2c
[scala-adt]: https://alvinalexander.com/scala/fp-book/algebraic-data-types-adts-in-scala/
[monadic-laws]: http://en.wikipedia.org/wiki/Monad_(functional_programming)#Monad_laws