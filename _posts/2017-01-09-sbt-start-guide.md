---
layout: post
title: "sbt Beyond the Defaults: Part I"
date: 2017-01-09 20:30:00-0500
categories: ["scala", "sbt", "functional programming"]
---

Welcome to **sbt beyond the defaults** series! This is my first post here and, as you know I'm a Scala developer (since mid 2015) and I feel that there's so much left to do and learn about the language and the ecosystem, but I've grasped the basics for getting things done, except for one thing: `sbt`.

For the vast majority of the Scala newcomers `sbt` is one of the greatest challenges (myself included). When you see things like `:=`, `+=`, `~=`, `+==`, `***`, `<<=` and so on it's inevitable to think about the *complexity* of this tool. And I won't deny it: **it's hard**. Not only because of the DSL but also for the lack of learning resources available out there (at least for grasping the basics), and for other reasons (I'll mention some of them):

* **Lack of concise documentation:** The official documentation is a good resource to start, but you have to invest time to understand the basics and start to do ~~copy-paste the configuration~~ your own code. `sbt` maintainers are working on a [revamp of the docs](https://github.com/sbt/website/pull/306) and it's truly an appreciated effort.
* **The API docs need some work:** You won't find detailed information in the [API Docs](http://www.scala-sbt.org/0.13/api/#package) and that's a drawback because it makes the learning curve way more steep.
* **Lack of *newbie* resources:** Well, *sort of*. There are [very](https://www.youtube.com/watch?v=V2rl62CZPVc) [good](https://www.youtube.com/watch?v=X6CnYQDL9Eg) [talks](https://www.youtube.com/watch?v=8aCLutYgRLA) about `sbt` but (and this is a personal opinion) none of these are focused for novices. Yeah, sure you need to understand `Scopes`, `Keys` and the concepts around the tool, but how do you focus the knowledge in a *step-by-step* manner to ease the process for newcomers? That's the point of writing this series (and I hope to achieve that).

[You'll see criticism](http://dimafeng.com/2017/01/04/sbt/) [around](https://www.reddit.com/r/scala/comments/5a6muj/sbt_makes_me_want_to_give_up_scala/) [the tool](http://imranrashid.com/posts/scala-wrong/), but the idea here is to show that it's possible to create a simple *Getting Started* guide to understand the basics and then, if you like, continue your learning process based on the official documentation about `sbt`.

## This isn't a criticism post, so let's get started!

I've talked about a little bit of history and the actual drawbacks. Having a technical debt with myself about this topic, I'm motivated to write a series of posts (I don't know exactly how many of them) trying to explain my learning path about `sbt` in a straightforward way.

**But, before that, I have to make this disclaimer:**

>**English is NOT my first language:** but I've decided to let the shame apart and write in English because of the reasons noted [here](/about). If you find typos or grammar mistakes I appreciate if you can note those in the comments. With this, I improve my writing skills and also improve the content of the posts... :)

**And also:**

> *This series assumes that you have at least some experience with Scala and `sbt`. This means that you have created a very simple 'Hello World' and have runned it. That's all we need here.*

That said, let's begin!

## Some things to know about `sbt`

* **`sbt` works by convention:** sbt respects the [CoC principle](https://en.wikipedia.org/wiki/Convention_over_configuration) as Maven / Gradle does, so it's expected (but not mandatory) to see things like the source directory hierarchy (`src/main/scala`. `src/test/scala`, `src/main/java`, `src/test/java`), your main build definition called `build.sbt` and so on. You can see more about it [here](http://www.scala-sbt.org/1.0/docs/Hello.html).
* **`sbt` is a Task Engine:** This is a very interesting definition, because you'll see that almost everything in the tool is based on how we program tasks to run, how we chain them together to achieve some sort of computation or side effect (that sounds like a pipeline of tasks). This approach was addressed by [James Roper](https://twitter.com/jroper) of Lightbend [in this post (it's a must-read!)](https://jazzy.id.au/2015/03/03/sbt-task-engine.html).
* **Scopes and Settings:** `sbt` relies heavily in two concepts that we'll see in a detailed manner in other posts: Scopes and Settings. For now, we'll going to see a brief introduction about what Settings are.

## Settings for our builds

If you are familiar with other build tools such as Maven or Gradle, you'll know that all your app lifecycle can be handled with a series of tasks like `build`, `compile`, `test` and many more. In the case of Maven, you can define and customize the specific behaviour of these steps in the POM file of your project. Being this descriptor a XML file there is plenty of settings available for you, in form of XML tags.

In our case, `sbt` defines a Map of keys that describes all the possible settings that we need to describe our project (think as the metadata of our project: the name, the organization, the version) and also specific configuration that affects our build (more on this later).

Unfortunately, all available keys are not in the docs but it can be seen [here](https://github.com/sbt/sbt/blob/1.0.x/main/src/main/scala/sbt/Keys.scala) with its description. Also you can define your own keys but for the sake of brevety this topic will be approached in other chapters.

This is the very general definition of a Setting and i'm going to explain it in depth in subsequent chapters.


## About "signs" and keywords

If you use [Lightbend Activator](https://www.lightbend.com/activator/download) for scaffold a very basic Scala project you'll see that the `build.sbt` generated by this tool contains something like this:

```scala
name := """my_app_name"""

version := "1.0"

scalaVersion := "2.12.1"

// Change this to another test framework if you prefer
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```

This sort of configuration works without any issue for a very simple application. But if you're like me, maybe you are wondering:

* Why (if you are seeing the `build.sbt` in your favourite IDE) `name`, `version`, `scalaVersion` and `libraryDependencies` are keywords?
* Why a simple assign statement (`=`) doesn't work and instead you have to do that with `:=`?
* What the hell is `+=`?

Let's start by saying that we're seeing one of two possible ways to define our builds, [being the other one](https://stackoverflow.com/questions/18000103/what-is-the-difference-between-build-sbt-and-build-scala) with `.scala` files, but this alternative was [deprecated in 0.13.12](https://github.com/sbt/sbt/pull/2530) and it's encouraged to write our definitions in `.sbt` files. Let's see step by step the answers for our questions.

#### Why  `name`, `version`, `scalaVersion` and `libraryDependencies` are keywords?

Remember our brief explanation about Settings? Well, here is the answer: **these keywords are settings!** In our build definition (`.sbt` build style) the classes and objects that sbt needs to work are automatically imported, so you don't need to explicitly import them (thing that you have to do with the `.scala` build style). The implied imports in a `build.sbt` file are:

```scala
import sbt._
import Process._
import Keys._
```

With this, all the sbt keys are available for our builds. In this ~~naive~~ straightforward example `name`, `version`, `scalaVersion` and `libraryDependencies` are not explicitly useful for us (but for `sbt`!) because we haven't defined any task that use this settings yet.

#### Why a simple assign statement (`=`) doesn't work and instead you have to do it with `:=`?

For now, we only need to know that sbt has a map of the Settings with their keys and respectively values. This setting keys aren't variables, so they have to be assigned / updated through certain functions and, in this case we can do it with `:=`.

Wait a minute... is `:=` a method? The answer is: yes! Remember that, in Scala we can evaluate a function that receives only one parameter without the dot notation. So, this:

`name := """my_app_name"""`

Is equivalent to:

`name.:=("my_app_name")`

This method assigns or updates a value to the specified key as described [here](http://www.scala-sbt.org/1.0/docs/Basic-Def.html#How+build.sbt+defines+settings).

One very important thing to note is that Settings has their own type: `SettingKey[T]` and `:=` function can only evaluate and return this data type. So, if you try to do this, for example:

`name := 123`

It won't compile, because the Setting `name` is defined as `SettingKey[String]`. Also note that this function will only return `SettingKey[T]`.

If you feel curious, this is the signature of the function:

`final def :=(v: T): Setting[T] = macro std.TaskMacro.settingAssignMacroImpl[T]`

And it can be found [here](https://github.com/sbt/sbt/blob/1.0.x/main-settings/src/main/scala/sbt/Structure.scala#L39) (among other *cryptic* ones).

#### What the hell is `+=`?

`libraryDependencies` is the setting that specifies the managed dependencies of our project. As you can see in the example we have three *Strings* (let's call them in this way for sake of simplicity, we'll return to this later) separated by percentage signs (%). For now, you can see that we have only one dependency: ScalaTest. But, how about having more of them as our application grows? Well, this setting can be specified as follows:

**One-liner dependency**

```scala
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```

**Multiple dependencies in different lines**

```scala
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
libraryDependencies += "org.typelevel" %% "cats" % "0.8.1"
```

What do you think? It seems like an append, right?

In this case, `libraryDependencies` is a `Seq` containing all the archetypes of our dependencies. `+=` allow us to append more entries to the sequence. Note that we're appending objects to the internal structure that this SettingKey has. We're not using `:=` function because we're not assigning an entire `Seq(dependency, dependency, ...)` but an individual `dependency` to the `Seq`.

In fact, we can assign an entire `Seq` as follows:

```scala
libraryDependencies ++= Seq(
"org.scalatest" %% "scalatest" % "3.0.1" % "test".
"org.typelevel" %% "cats" % "0.8.1")
```

Please note that this is a simplified version of the multiple dependencies in different lines. Also note that if we want choose this option we have to use the operator `++=` (assign an entire `Seq` of objects to the Setting) instead of `+=` (assign an object to the `Seq`).  

As earlier, if you feel curious about these functions, this is the signature for both of them:

```scala
final def +=[U](v: U)(implicit a: Append.Value[T, U]): Setting[T] = macro std.TaskMacro.settingAppend1Impl[T, U]

final def ++=[U](vs: U)(implicit a: Append.Values[T, U]): Setting[T] = macro std.TaskMacro.settingAppendNImpl[T, U]
```

And you can find out more about it [here](http://www.scala-sbt.org/1.0/docs/More-About-Settings.html#Appending+to+previous+values%3A++and) and in [this StackOverflow answer](https://stackoverflow.com/a/23693921/478030).

# Summary

This is the first post of the **`sbt` beyond the defaults** series. I tried to keep it as concise as possible, because we'll see the concepts around the sbt basics in more depth.

Here we've talked about the general impression that `sbt` has among developers, its general critics and then we've started to grasp some very basic definitions (like the `SettingKey` map that we'll see in detail in the next post). Then, we've looked a very simple build definition, explaining the ***magic*** syntax around it in detail.

It seems like a very basic approach to this topic but I think that it's neccesary to explain this straightforward example to move on with more detailed (but basic) aspects of `sbt`.
