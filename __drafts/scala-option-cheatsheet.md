---
layout: post
title: "From Scala Option to Monad transformers"
description: "Scala Option cheatsheet"
category: scala
tags: [Scala]
---

The Scala [Option type](https://www.scala-lang.org/api/current/scala/Option.html) is one of the easiest and most useful things that Scala offers to handle optional values. The official docs state:

>  The most idiomatic way to use a `scala.Option` instance is to treat it as a collection or monad and use `map`, `flatMap`, `filter`, or `foreach`

But very often a developer ends up writing several lines of pattern matching instead of choosing a quicker and more concise way of handling the common cases that involve `Scala.Option`. 

As a matter of fact, we can handle an Option using a higher-order function in the vast majority of cases, which is something that we're going to see in details down below.

```scala
/** A higher-order function that can be used to substitute pattern matching */
def optional[T, U](o: Option[T])(none: => U, some: => T => U): U = ???
```

Once we have an Option[T] `o` we define what happens when `o` is either `None` or `Some`, given that we'll end up with an object of type `U` eventually. The most common use case is `Option.map()`:

```scala
def map[T, U](o: Option[T], f: T => U) =
  optional(o, None, a => Some(f(a)))
```

In this post, I'll try to enlist a series of common examples of pattern matching, which many developers might find themselves performing, followed by the use of a function that already exists on Option that encapsulates that given form of pattern matching.

---

Before starting with the examples let's assume that we'll start with an Option `val opt: Option[T]` and a generic function `f: T => U`

### flatMap

```scala
opt match {
  case Some(x) => f(x)
  case None    => None
}
```

Now we want to apply `f()` to the optional contents of our Option, or just get None if the Option carries nothing. This is equivalent to:

```scala
opt.flatMap(f(_))
```

### map

If we want to apply `f()` while keeping our `Option` wrapper we can use `map()`

```scala
opt match {
  case Some(x) => Some(f(x))
  case _       => None
}
```

which becomes:

```scala
val y: Option[U] = opt.map(f(_))
```

### foreach

```scala
opt match {
  case None    => ()
  case Some(x) => 
    f(x)
    ()
}
```

This code is equivalent to:

```scala
opt.foreach(f(_))
```