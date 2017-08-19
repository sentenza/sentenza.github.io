---
layout: post
title: "Work with Scala using sbt and Spacemacs"
description: "How to be productive with Scala using lean tools"
category: Programming
tags: [Scala, emacs]
---
{% include JB/setup %}

Start programming and experimenting with a functional programming language like Scala might not seem simple at first sight. So,
to get your feet wet it's useful to have the very essential in a single place. I started writing this post in the form of
a gist, but then I realized that maybe it was better to share it with the community. 

<img src="http://www.scala-sbt.org/assets/typesafe_sbt_svg.svg" height="100" width="100" align="middle">

`sbt` (aka _Simple Build Tool_) is a general purpose build tool written in Scala for JVM developers. It took advantage of previous ideas developed by other similar tools like Maven, Gradle and Ant.

1. Default project layouts
2. Built-in tasks
3. Plugin architecture
4. Declarative Dependency management
5. Code over Configuration: A DSL for build tool
6. Interactive
7. Scala REPL integration

In order to have a better understanding of the main tool you'll use programming with Scala you must read:

- [SBT: The missing tutorial](https://github.com/shekhargulati/52-technologies-in-2016/blob/master/02-sbt/README.md)
- [The sbt reference manual](http://www.scala-sbt.org/0.13/docs/index.html)

<!--more-->

## Project scaffolding (doing it in the right way)

The best way to organize your codebase, using **sbt**, is to follow [the same folder structure used by Maven](http://www.scala-sbt.org/0.13/docs/Directories.html):

```
src/
  main/
    resources/
       <files to include in main jar here>
    scala/
       <main Scala sources>
    java/
       <main Java sources>
  test/
    resources
       <files to include in test jar here>
    scala/
       <test Scala sources>
    java/
       <test Java sources>
```

### Create a new project using sbt

At first, you've to create the new project folder (e.g. `myproject`). Then, put inside the new folder a file named `build.sbt`, which contains the build settings that sbt will use. to house the build script ([how build.sbt defines settings](http://www.scala-sbt.org/0.13/docs/Basic-Def.html#How+build.sbt+defines+settings)). This file, at the very beginning, will look like the following:

```scala
name := "myproject"
version := "0.1.0"
scalaVersion := "2.12.2"
sbt.version=0.13.16
```
where `:=` is a function defined in the `sbt` library. It is used to define a setting that overwrites any previous value without referring to other settings. In order to define the version of Scala that sbt will pick up it's possible to define a specific `scalaVersion`, but before doing so it's better to verify your environment configuration and the current version of your Scala compiler (`scala -version`). 

> If the Scala version is not specified, the version sbt was built against is used. It is recommended to explicitly specify the version of Scala.  
> Please note that because `compile` is a dependency of `run`, you don’t have to run `compile` before each `run`; just type `sbt run`.

@see [sbt documentation](http://www.scala-sbt.org/0.13/docs/Howto-Scala.html)

`sbt.version` [specifies the target sbt version](http://www.scala-sbt.org/release/docs/Basic-Def.html#Specifying+the+sbt+version) that our build will use. If the required version is not available locally, the sbt launcher will download it for you. If this file is not present, the sbt launcher will choose an arbitrary version, **which is discouraged because it makes your build non-portable**.  To retrieve the essential information about sbt (also see [this SO answer](https://stackoverflow.com/a/8462854/1977778)) you're running on you can use the following command:

```
sbt about

[warn] No sbt.version set in project/build.properties, base directory: /tmp
[info] Set current project to tmp (in build file:/tmp/)
[info] This is sbt 1.0.0
[info] The current project is {file:/tmp/}tmp 0.1-SNAPSHOT
[info] The current project is built against Scala 2.12.3
[info] Available Plugins: sbt.plugins.IvyPlugin, sbt.plugins.JvmPlugin, sbt.plugins.CorePlugin, sbt.plugins.JUnitXmlReportPlugin, sbt.plugins.Giter8TemplatePlugin
[info] sbt, sbt plugins, and build definitions are using Scala 2.12.3
```

What's more, in the _interactive mode_ `sbt` has a list of useful commands that guide you in the day to day work like `help` and `tasks`. One of the most useful things of sbt is the automatic and continous execution of a task fired by the changes saved to a source file you're editing. To trigger this kind of behaviour a task must be prepended with `~`. So, to execute all the tests that either failed during the previous execution or whose transitive dependencies changed the task is `~testQuick`, to stop its execution just hit _Enter_.

#### Add ScalaTest
Starting from sbt 0.10 it's possible to add the [ScalaTest framework](http://doc.scalatest.org/3.0.1/#org.scalatest.tools.Framework) to a sbt project adding the following line to the `build.sbt`:

```scala
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```

- [Using ScalaTest with sbt](http://www.scalatest.org/user_guide/using_scalatest_with_sbt)

#### Start simple using the `hello-world` template
 To start in the fastest way possible using `sbt` there is a `hello-world` template (the [full tutorial](https://www.scala-lang.org/documentation/getting-started-sbt-track/getting-started-with-scala-and-sbt-in-the-command-line.html)) that can be imported just using `sbt` itself:

```bash
$ sbt new scala/hello-world.g8
```
@see [Getting started with Scala and sbt in the CLI](https://www.scala-lang.org/documentation/getting-started-sbt-track/getting-started-with-scala-and-sbt-in-the-command-line.html)

## Resources

### IDE and its configuration

- [Emacs for Scala development](http://www.rabbitonweb.com/2016/01/31/my-emacs-for-scala-development-part-1/)
- [Spacemacs and the Scala layer](https://github.com/syl20bnr/spacemacs/tree/master/layers/%2Blang/scala)
- [Spacemacs and ENSIME](http://spacemacs.org/layers/+lang/scala/README.html)
