---
layout: post
title: "sbt Beyond the Defaults: Part II - Keys"
date: 2017-01-22 10:50:00-0500
categories: ["scala", "sbt", "functional programming", "sbt beyond the defaults"]
---

Welcome to another post of the _sbt Beyond the Defaults_ series. If you missed the purpose of this series you can read the first part [here](sbt-start-guide).
We have seen some basic concepts briefly explained about **sbt** and I talked about `Settings`. In this part we'll take a look in depth about this concept and its generalization with `Task`.

## What we've seen so far about Settings

Settings are the different configuration keys that we can find in `sbt` for program our build definition. This concept would be the main reasoning unit in sbt (at some extent). We also learned that keys like `name`, `version` and `scalaVersion`, for example, are Settings of our project and these keys are stored in an immutable map available in our `build.sbt` and that we can assign values to the keys with the function `:=`.
That said, let's dive into the details of this concept!

## Setting Keys

I mentioned earlier that `Settings` is an immutable map. Following the functional paradigm definition it means that with every assignment instruction the _entire_ map is recomputed, obtaining a new one. An advantage of the sbt definition of this is that you can access the keys directly without any sort of complicated syntax (like Java-ish `Map.get` or so). You have seen this throughout the last post.
If you open a `sbt` shell in your project and type one of these keys you'll get the value associated with them. I will take the sbt definition of the previous post:

```scala
name := """my_app_name"""

version := "1.0"

scalaVersion := "2.12.1"

// Change this to another test framework if you prefer
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"

```
Now, if you do this in a console (in your project folder):

```shell
$ sbt
> name
```
You'll get:

```shell
> [info] my_app_name
```

Remember, I provided the detail about the structure being a _Map_ but you don't have to worry about that. You only have to think in terms of _Keys_ (or _Keywords_ if you want).

### Settings are typed!

We have seen this in the previous post, but I want to emphasize that you can have any sort of Keys with their respective types, no matter if a Key stores an _Object_ or merely a primitive value (like an `Int` or a `String`). You can see, for example, the types of the `name`, `version` and `scalaVersion` keys here (Taken from [Keys.scala](https://github.com/sbt/sbt/blob/1.0.x/main/src/main/scala/sbt/Keys.scala)):

```scala
val name = SettingKey[String]("name", "Project name.", APlusSetting)
val version = SettingKey[String]("version", "The version/revision of the current module.", APlusSetting)
val scalaVersion = SettingKey[String]("scala-version", "The version of Scala used for building.", APlusSetting)
```

For now, you can ignore the `APlusSetting` parameter. Other than that, you can see that `SettingKey` receives a type parameter and two arguments: the key name and the key description.
The mentioned keys are part of the standard set of predefined ones, but we can create our own as part of our builds. For this example, we are going to add a `buildDescription` to your project (in the `build.sbt` file):

```scala
val buildDescription = SettingKey[String]("buildDescription", "Description of this project")

buildDescription := "My SBT Learning Project! Yay!"
```

Note that I have assigned a value to the Key, so if I do this:

```shell
$ sbt
> buildDescription
```

I'll get:

```shell
> [info] My SBT Learning Project! Yay!
```

### Lifecycle of `SettingKey[T]`

`SettingKey[T]` defines all the neccesary values that you would need in your build and they are loaded within project initialization. That means, when started, the keys and their values remains available during all the execution phase of the project (build). Note that the value of each key defined is computed _only once_. If you make any change to the value of any of your build settings you need to tell `sbt` to `reload` the project for the changes to take effect, like this:

```shell
$ sbt
> reload
```

## All in the world is not only about setting keys

We have seen in detail `SettingKey` and its behaviour, but your build is not only formed with settings, right? I mean, if you have, for example, your `pom.xml` file describing a Maven build you don't only have the settings about the project, but different **_tasks_** ([_Phases_ is the correct term in Maven](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)) that you would need at some point, depending on your build flow. In `sbt` the concept is abstracted as `Keys` and there are three types of them:

1. `SettingKey[T]`
2. `TaskKey[T]`
3. `InputKey[T]`

We've seen the usage and the details of `SettingKey[T]` and now we will move to `TaskKey[T]`.

## A Task Engine is made of Tasks, right?

In the previous post I mentioned the concept brought by James Roper about sbt being a _Task Engine_, so it's safe to say that one of the key components of the tool is the `TaskKey[T]`.
A `TaskKey[T]` is a type of key that holds a value, called a _task_ which can store the result of a computation or even manage a side effect (write to a file, to the SystemOut, to a Database, etc.).
One of the main differences between a `SettingKey` and a `TaskKey` is that the first one holds a value computed at the initialization of the project until its execution ends. Meanwhile, a `TaskKey` is a value that can be _re-computed_ as many times as we want (upon request).
Also note that a Setting doesn't have any external dependencies because they are _immutable_ values that doesn't change.

With Tasks, we could have as many dependencies as we want. These dependencies could be external (databases or files, for example) or even Settings in our build.

### We interact constantly with Tasks

If you have used `sbt` previously it's probable that you are familiar with _commands_ like `compile`, `update`, or `reload`. These _commands_ are Tasks and, in the case of the aforementioned ones, are default tasks that `sbt` uses for the purposes described by each one.

A task is defined with the following syntax:

```scala
val taskName = taskKey[T]("taskName", "taskDescription")
```

So, for example, `compile` task, bundled by default in sbt is described by the following signature:

```scala
val compile = TaskKey[CompileAnalysis]("compile", "Compiles sources.", APlusTask)
```

Once again, please ignore at this time the third argument (`APlusTask`). We will see that on detail in later chapters of this series. In this case, `compile` is a task that returns a `CompileAnalysis` object.

**Note that another way to define a task is by simply passing the description parameter. The name of the task will be the name given to the variable, as follows:**

```scala
val dummyTask = taskKey[Unit]("A very dummy task")
```

### How can we define a custom task?

Let's see a very straighforward example: defining a Task for print the value _Hello, world_ in console. Don't worry! I'll expand the example to something _practical_ later.

In our `build.sbt` file:

```scala
lazy val ourHelloWorldTask = taskKey[Unit]("'Hello, World' task")

ourHelloWorldTask := {
	println("Hello, World!")
}

```

**Things to note here:**

* I've used `lazy val`. This is because we want to avoid _initialization issues_. That means that if some task depends on another we don't want to have an unresolved dependency problem because of the order in which the task were loaded (this is evaluated on project's load).
* The return type is `Unit` because the task is a merely side effect (it only prints a message to the standard output).
* I've used the `:=` function again because it works not only for values but for the statements that you would add to a task as well.

In a console, if you type:

```shell
$ sbt
> ourHelloWorldTask
```

You'll get:

```shell
Hello, World!
[success] Total time: 0 s, completed 21/01/2017 05:20:34 PM
```

What about printing the `name`, `version` and `scalaVersion` instead?

In that case I'm going to combine the Task with values from Settings as follows:

```scala
ourHelloWorldTask := {
  val theText = s"This project is called ${name.value}, " +
    s"with version ${version.value} and the Scala version used is ${scalaVersion.value}"
  println(theText)
}
```

**Things to note here:**

* There are _two_ statements inside the assignment block: the first one is the String interpolation, defined in a variable called `theText` in where I used the `name`, `version` and `scalaVersion` setting keys. You can have blocks of code inside the `:=` without any issue. The other one is the `println` function that outputs the string to the stdout.
* You need to use the `.value` function to get the _real value_ of a key. By definition, using only the key will get you a `SettingKey[T]`, so you need to access its value with the `value` modifier. In this case you're getting the `T` type of the key.

Again, if you re-run the task:

```shell
$ sbt
> ourHelloWorldTask
```
You'll get:

```shell
This project is called my_app_name, with version 1.0 and the Scala version used is 2.12.1
[success] Total time: 0 s, completed 21/01/2017 05:37:08 PM
```

## A Task with steroids

We've just seen the very basics about Tasks. Now consider that, for some reason we need some input provided by the user for the task to run. In that case we'll need an Input Task.

Again, the syntax is the same as the `SettingKey` and `TaskKey`, but basically an `InputTask[T]` makes use of a special feature of Scala called **[Parser Combinators](http://berniepope.id.au/docs/scala_parser_combinators.pdf)**. Being Parser Combinators a very broad topic itself I'm going to stick to the basics and I'll come back later to this.

I'm going to retake the latter example of printing out the `name`, `version` and `scalaVersion`, but this time the arguments will be provided by the user (I know, I know, it's a very naive test, but it's for the sake of learning! I promise that I'll retake this in depth in next posts).

So, We'll have:

```scala
import sbt.complete.DefaultParsers._

lazy val ourHelloWorldTask = inputKey[Unit]("'Hello, World' task")

ourHelloWorldTask := {
  val args: Seq[String] = spaceDelimited("<arg>").parsed
  println("This project name, project version and Scala version used is (respectively): ")
  args foreach println
}
```

**Things to note here:**

* Our task is now an `inputKey[Unit]`.
* This message is printed in the standard output: **_This project name, project version and Scala version used is (respectively)_**, followed by the three parameters that I provided to the task.
* `spaceDelimited` is a parser combinator that _parses_ the input arguments that I provided and stores it in a `Seq[String]`. It assumes that, as its name says, the parameters are separated by a space. 
* To use this parser combinator, it is neccesary to explicitely import the parsers bundled with sbt. That is the reason for seeing the line `import sbt.complete.DefaultParsers._`.

So, if we run the task:

```shell
$ sbt
> ourHelloWorldTask My_App 1.0 2.12.1
```

We'll get:

```shell
This project name, project version and Scala version used is (respectively):
My_App
1.0
2.12.1
[success] Total time: 0 s, completed 21/01/2017 06:55:11 PM
```

## Summary

We've seen `SettingKey` in depth. Also we learned that there are three types of Keys: `SettingKey`, `TaskKey` and `InputKey`. We reviewed the usage of a Task with (**InputKey**) and without (**TaskKey**) parameters.

It's important to note that we spotted the differences between a Setting and a Task and also we've seen their lifecycle. I tried to explain each of the concepts here with a set of very basic examples for illustrate how does that works.

I'm trying to keep this posts short and concise because I don't want you to saturate with lots and lots of information. Some topics are on hold:

* Task chaining and dependency between tasks.
* Task modification.
* Task overriding.
* A basic lesson about parser combinators with Input Tasks.

I'm going to try to cover these topics in the next posts. For now I can say that the next one will be about Scopes, a key concept of `sbt`. After that we can move on this pending topics.

Thanks for reading! See you in the next chapter! :)