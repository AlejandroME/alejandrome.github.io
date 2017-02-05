---
layout: post
title: "sbt Beyond the Defaults: Part III - Scopes"
date: 2017-02-05 15:15:00-0500
categories: ["scala", "sbt", "functional programming", "sbt beyond the defaults"]
---

Welcome to another post in _sbt Beyond the Defaults_ series! Today I am going to cover another _key_ concept of sbt: **_Scopes_**. In previous chapters of this series we learned the different types of keys and their usage in a very straightforward way. Also we have seen the lifecycle of each one of the keys, how to define our own and how to use them.

Now I am going to show you the different scopes that covers each key and how to understand each one of them.

## Facts about `Keys`

I will share some aspects about Keys.

* Each key can have more than one value. That is, for every key there is an associated value in a **_context_**.
* A key could have dependencies on other keys.
* Based on the **_context_** hierarchy, a key could have a fallback value.

Note that I have mentioned the word _context_ twice. That context is called a **scope**.

## Scopes

A scope is the _context_ in which a specific key will apply. This is a way of separate different behaviors that a key would have. For instance: when you want to apply a different compilation process to the sources of your project and the tests.
That example could be extended if you want to implement a static code analysis tool like [scalastyle](http://www.scalastyle.org/) or [scapegoat](https://github.com/sksamuel/scapegoat) in your project and you want to throw a compilation error if some of your configured rules were violated during the process. But, you don't want to apply the same behaviour in the same phase (compilation) of your tests.

In that case, the `compile` taskKey is the same (syntactically) for compilation and testing, but its _value_ is different for each of the scopes.

We are going to see more on this later.


## Scope Axes

`sbt` has three scope types (called _axes_) in where each key can be defined with its own value:

* Project Axis.
* Configuration Axis.
* Task Axis.

But before to take a deep look to each of them we are going to see a way to fully understand the hierarchy of a key.

## `Inspect` me!

`sbt` has a built in task called `inspect` that let us see all the details of a specific key, thus you can watch its description, dependencies and definition.

I am going to take the same basic sbt example that I've previously used for demonstration purposes:

```scala
name := """my_app_name"""

version := "1.0"

scalaVersion := "2.12.1"

// Change this to another test framework if you prefer
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```

Remember that this is the very basic definition of a `build.sbt` file, provided by Lightbend Activator. If you remember the types of keys that we've seen in the [previous chapter](sbt-beyond-the-defaults-keys) you'll notice that this file only has `SettingKeys` (`name`, `version`, `scalaVersion` and `libraryDependencies`), so let's try inspecting one of them:

```scala
$ sbt
> inspect name
[info] Setting: java.lang.String = my_app_name
[info] Description:
[info] 	Project name.
[info] Provided by:
[info] 	{file:/Users/AlejandroE/Documents/my_app_name/} my_app_name/*:name
[info] Defined at:
[info] 	/Users/AlejandroE/Documents/my_app_name/build.sbt:1
[info] Reverse dependencies:
[info] 	*:description
[info] 	*:onLoadMessage
[info] 	*:projectInfo
[info] 	*:normalizedName
[info] Delegates:
[info] 	*:name
[info] 	{.}/*:name
[info] 	*/*:name
```

You'll see this output in your console. What does that means?

* The _Setting_ describes the type of the key and its value. In this case it's a Setting, of type `java.lang.String` with value `my_app_name`.
* The _Description_ shows the description value of the key. Remember: when you are defining a key you can provide a description (we've seen this in the previous chapter). In this case, this value is taken [from the defaults](https://github.com/sbt/sbt/blob/1.0.x/main/src/main/scala/sbt/Keys.scala#L282).
* The _Provided by_ shows which key defines its value. We're going to see that the asterisk in `*:name` has an important meaning. For now, I can say that this key is provided by the _global defaults_.
* The _Defined at_ shows where the key is defined at. In this case it's in our `build.sbt` at line 1.
* The _Reverse dependencies_ shows what other keys uses this specific key. In this case `name` is used by `description`, `onLoadMessage`, `projectInfo` and `normalizedName`.
* The _Delegates_ shows you in where `sbt` will look in case of not finding the value of this key in the `build.sbt` file.

As you can see, every key defined in sbt has its properties and its defined scope and dependencies. In the previous example we have looked at a Setting Key, but what happens when you look at a Task Key or even an Input Key? Let's see.

```scala
$ sbt
> inspect compile
[info] Task: sbt.inc.Analysis
[info] Description:
[info] 	Compiles sources.
[info] Provided by:
[info] 	{file:/Users/AlejandroE/Documents/my_app_name/} my_app_name/compile:compile
[info] Defined at:
[info] 	(sbt.Defaults) Defaults.scala:280
[info] Dependencies:
[info] 	compile:manipulateBytecode
[info] 	compile:incCompileSetup
[info] Reverse dependencies:
[info] 	compile:discoveredSbtPlugins
[info] 	compile:discoveredMainClasses
[info] 	compile:products
[info] 	compile:printWarnings
[info] Delegates:
[info] 	compile:compile
[info] 	*:compile
[info] 	{.}/compile:compile
[info] 	{.}/*:compile
[info] 	*/compile:compile
[info] 	*/*:compile
[info] Related:
[info] 	test:compile
```

You can see that we've inspected the `compile` task and, in fact, sbt are telling you that it's a `Task` of type `sbt.inc.Analysis`. Also note that the structure of the inspection is almost the same as the previous inspection with `name`. Now we are going to see the different scopes and how relate to this structure.

## Detail of Scope Axes

### Project Axis

With sbt, you can have a multi-project structure. That is, as its name says, subprojects inside a build definition. In our case, we are working with a single project definition, so the keys that we define are by default scoped to our project (unless we specify the scope).

If we choose to have a multi-project definition we can scope our keys by a single subproject or even to the entire build and that counts as a project scope. The latter example requires to understand [the multi-project structure](http://www.scala-sbt.org/1.0/docs/Multi-Project.html), so I won't be covering any more details on this at the moment for the sake of clarity. 

Retaking our example, if I do this:

```scala
$ sbt
> my_app_name/name
```

I'll get:

```scala
> [info] my_app_name
```

Note that I've prefixed the `name` key with the project name and a forward slash. With that syntax I'm telling sbt to _show me the value of `name` in the project axis_. Remember that we have the value `my_app_name` at that scope because we defined it in our build.sbt.

But, what would happen if I remove the `name` key from our build.sbt and then I try to get the value associated to the key in the project scope? Let's see

```scala
//name := """my_app_name""" - I commented this line

version := "1.0"

scalaVersion := "2.12.1"

// Change this to another test framework if you prefer
libraryDependencies += "org.scalatest" %% "scalatest" % "3.0.1" % "test"
```

```scala
$ sbt
> my_app_name/name
> [info] my_app_name
> 
> name
> [info] my_app_name
```

Please note that I've asked sbt for the key `name` in two ways: prefixing the project name (project scope) and with the _global scope_. Both of them returned the name of my project, but why? I have commented the key value, haven't I?

Yes, but remember

1. Your project has a name, even if you don't specify it in the `build.sbt`. Having said that, `sbt` will look that value, if you don't tell it one associated to that key, finding the folder name.
2. We have seen with the `inspect name` command that the key has _Delegates_, that means, if you don't specify the value the tool will look at `*:name`, `{.}/*:name` and `*/*:name` scopes. Remember that the `*` prefix means _global_ (the entire build).


### Configuration Axis

The configuration axis defines the possible settings that a task need for its execution. For instance, take a look at `compile`: it is a configuration axis that has a task with the same name as follows.

```scala
$ sbt
> compile:compile
[success] Total time: 0 s, completed 5/02/2017 11:54:50 AM
```

When you type `compile` (without the configuration axis) you are actually typing an alias of `compile:compile`. What does `compile`, as a configuration scope has? Let's see.

```scala
$ sbt
> inspect compile:sources
[info] Task: scala.collection.Seq[java.io.File]
[info] Description:
[info] 	All sources, both managed and unmanaged.
[info] Provided by:
[info] 	{file:/Users/AlejandroE/Documents/my_app_name/} my_app_name/compile:sources
[info] Defined at:
[info] 	(sbt.Defaults) Defaults.scala:200
[info] Dependencies:
[info] 	compile:unmanagedSources
[info] 	compile:managedSources
[info] Delegates:
[info] 	compile:sources
[info] 	*:sources
[info] 	{.}/compile:sources
[info] 	{.}/*:sources
[info] 	*/compile:sources
[info] 	*/*:sources
[info] Related:
[info] 	test:sources
```

In this case we've inspected the task `sources` of the configuration axis of `compile`. That means that when you are going to compile the project's code `sbt` will perform a series of task executions that are dependencies for the whole compilation process. In this case, `sources` returns the sequence of files to be compiled.

The concept of configuration, [as noted by the official sbt documentation](http://www.scala-sbt.org/1.0/docs/Scopes.html#Scoping+by+configuration+axis) comes from the same term from [Ivy](https://ant.apache.org/ivy/) and the [Dependecy Scopes of Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope).

### Task Axis

The last axis is related to task. Imagine a _task chaining_ operation in where one task depends on the outcome of another one. It doesn't necessarily depends on its result, but on the execution of the latter.

Taking the example of the official documentation, the predefined task `packageSrc` has a setting called `packageOptions` that modifies the outcome of the task. That is, if you want to package your source code (`packageSrc`), the binaries of your application (`packageBin`) or the documentation (`packageDoc`) of the application. These last ones belongs to the set of options for that task. Let's inspect it!

```scala
$ sbt
> inspect packageSrc:packageOptions
[info] Task: scala.collection.Seq[sbt.PackageOption]
[info] Description:
[info] 	Options for packaging.
[info] Provided by:
[info] 	*/*:packageOptions
[info] Defined at:
[info] 	(sbt.Defaults) Defaults.scala:624
[info] Delegates:
[info] 	{.}/packageSrc:packageOptions
[info] 	{.}/*:packageOptions
[info] 	*/packageSrc:packageOptions
[info] 	*/*:packageOptions
[info] Related:
[info] 	compile:packageSrc::packageOptions
[info] 	test:packageBin::packageOptions
[info] 	test:packageSrc::packageOptions
[info] 	*/*:packageOptions
[info] 	compile:packageBin::packageOptions
```

As you can see, `packageOptions` is a task that returns the sequence (enumeration) of the possible packaging options that you have. This is like a _modifier_ for changing the behavior of the task.

## Structure of the Axes

We've seen different syntax for illustrate the different scopes, but it's good to know that `sbt` has a standard one for parsing the right value according to the scope accessed as follows:

```scala
{<build-uri>}<project-id>/config:intask::key
```

Where

* `{<build-uri>}<project-id>` is the project axis. Remember that I have used this syntax previously for showing the `name` key of the project.
* `config` is the configuration axis.
* `intask` is the task axis.
* `key` is the key accesed through the scope that we're specifying.

Remember that, if some of these parts are not specified to sbt it will assume the default scope for that axes.
For instance:

```scala
$ sbt
> show my_app_name/compile:compile::sources
[info] * /Users/AlejandroE/Documents/my_app_name/src/main/scala/Domain.scala
[success] Total time: 0 s, completed 5/02/2017 01:44:55 PM
```

In this case I told sbt to show me the settingKey `sources` that is scoped by my project in the compile task of the compile configuration scope. The outcome is the only `.scala` file that I have in my project (`Domain.scala`).

## Summary

We've seen the three scopes provided by sbt: configuration, project and task. Also we've learned how to identify its hierarchy through the `inspect` command.
I brought an example of each of them and also I described the information provided by the inspection of the keys.

This chapter is fundamental for the next series. We've learned the very basics of `sbt` and I think that we are ready for more _practical_ posts. So, please stay tuned for the next one!

Finally, a very quick recap of this topic was done in an excellent answer by [Jacek Laskowsky](https://github.com/jaceklaskowski) [in StackOverflow](https://stackoverflow.com/a/26001025/478030).

Again, as usual, thank you for reading! See you in the next one :)