---
layout: post
title: Functional approaches: Reader, algebra and interpreter
date: 2017-07-03 17:30:00-0500
categories: ["cats", "reader", "ddd", "interpreter", "algebra", "functional patterns", "scala"]
---

I develop in Scala since 2015 and the current project in which I am assigned in I found a big application that has 4+ years of development. Sadly, because of multiple reasons (one of the first projects developed in Scala in the company, developers that were learning the language and developing the current product at the same time, developers that tried to do _Imperative Scala_ or _Java in Scala_ and so on) we have several microservices that takes little or none advantage from the functional capabilities of Scala.

Recently we needed to create a new microservice and I am one of the main developers, so, because of its nature I have decided to put into practice some concepts that I learned from the excellent [Functional and Reactive Domain Modelling](https://www.manning.com/books/functional-and-reactive-domain-modeling) by [Debasish Ghosh](https://twitter.com/debasishg). Also I have seen this opportunity a new way to teach ever member of my team how can we make well-designed, maintainable and readable software.

So, let's go!

## Before we start...

I created a _toy project_ called **ReaderM** that you can find [here](https://github.com/AlejandroME/ReaderM). It consists of a very basic API for managing my music tastes: classify my favourite artists, albums, bands and songs. In this stage of development you can see it as a very basic CRUD application with operations for querying, storing and updating data. The current implementation is made only for the Artists entity and it will be extended to the rest of the entities of the model.

If you're asking yourself: why that model/use case? It's because of my lack of imagination :sweat_smile:. I didn't wanted to reuse the same examples (inventory, coffee shop, online store and so on). It is a pretty straightforward model and I hope to illustrate the concepts of this post with it.

You can clone the project and follow the instructions of the readme for setting up the application.

One last thing about this: the set of technologies chosen for this project will let me write another couple of posts demonstrating its usage, but, for the scope of this post let's concentrate on some patterns that I'll like to introduce here.

## From Algebras and Interpreters...

Debasish, in his book, talks about the importance of an "Algebraic" design and how we can gain three characteristics in our application:

* **Loud and clear design.**
* **Compositionality.**
* **Verifiability.**

Citing the section 3.1 of his book:

> A module, as defined in a functional domain model, is a collection of _functions_ that operates on a set of _types_ and honor a set of _invariants_ known as the _algebra_ of the module. In mathematical terms, this is known as the _algebra_ of the module.

In this case, the algebra is our _published contract_ of the domain that we are modeling. This contract exposes the operations related to our domain model. In our case, we expose the algebra of the operations that we can do with our **Artists**. 

We'll see more details shortly.


## ... To Reader

Reader monad is not new. We'll explain it in a moment, but first, I like to reference a couple of excellent talks by [Rúnar Bjarnason](https://twitter.com/runarorama) about it and its usage for Dependency Injection:

* [Lambda: The Ultimate Dependency Injection Framework](https://yow.eventer.com/yow-2012-1012/lambda-the-ultimate-dependency-injection-framework-by-runar-bjarnason-1277)
* [Dead-Simple Dependency Injection](https://www.youtube.com/watch?v=ZasXwtTRkio)

I strongly encourage you to take a look to these talks. These are pretty short (~30 mins) and explain how to use this monad for the purpose that I've used it here.

## The Algebra

Our Artist domain is composed of certain operations in which we can rely on to manage our aggregate. Its published contract is as follows:

```scala
trait ArtistService[T] {

  def listArtists(): Reader[ArtistRepository, List[T]]
  def retrieveArtistById(id: Int): Reader[ArtistRepository, Option[T]]
  def retrieveArtistByName(name: String): Reader[ArtistRepository, Option[T]]
  def insertArtist(artist: T): Reader[ArtistRepository, Int]
  def updateArtist(artist: T): Reader[ArtistRepository, Int]

}

```

Let's take a detailed look at certain characteristics of this algebra:

1. **Trait parametrization:** you can see that our algebra is parametrized on `T` type. It means that the concrete implementation of this service will need to explicitely provide a type that suites the definition of every operation. In our case there's only one, but it could be more, depending of the needs of your algebra. What can we gain with this? Basically two things: making our contract explicit and preventing leakages. About the last statement: our implementation needs to attach to a set of invariants, no matter the concrete implementation.
2. **Compositionality:** you can add new operations to your algebra that could be implemented in terms of existing operations. For example: I can implement `insertArtist` looking for an `Artist` by name and if I couldn't find it then create it. For that purpose I can use the mentioned operation with `retrieveArtistByName`. I'm composing the first operation taking advantage of the latter.
3. **Abstraction over evaluation:** one thing that we tend to think when we don't have functional background is that every operation must produce a value. In FP, we appreciate to have an abstraction and rationalize in terms of it. That is, if `listArtists` _"returns"_ a `Reader[ArtistRepository, List[T]]`, I could use that monadic context for composing other operations that returns the same container. I don't have the value that `listArtists` is intended to produce (`List[T]`), but instead I have an abstraction that is way more powerful. More on this later.

As you can see, we have an Algebra that explicitely defines what its operations are, what their evaluation types are and in which types it relies on.

Using types that can by composed (a.k.a _Monads_) like `Try`, `Future`, `Option`, `Either` and so on you can compose these "small operations" to form larger operations or programs that are evaluated for producing a value. That is one of the main points for having an algebra and it's the core of the _abstraction over evaluation_.

Let's see that with an example: suppose that when I use `insertArtist` the operation returns an `Int` with the internal ID assigned in the database to that artist and I want to make an operation that, if the insertion succeeds returns the artist inserted. Please detail the return types of these two operations:

```scala
def insertArtist(artist: T): Reader[ArtistRepository, Int]
def retrieveArtistById(id: Int): Reader[ArtistRepository, Option[T]]
```

We have a _monad_ (`Reader`) that can be composed; also we have two operations potentially composable, because, in the order of actions that we need to execute, the evaluation of `insertArtist` produces an `Int` (the artist ID assigned in the DB) that can be used as the parameter `id` for the `retrieveArtistById` operation that finally yields an `Option[T]` with the artist information when evaluated.

So, our new operation will look like this:

```scala
def insertAndRetrieve(artist: T): Reader[ArtistRepository, Option[T]] = {
    for {
      id     <- insertArtist(artist)
      artist <- retrieveArtistById(id)
    }
      yield artist
  }
```

Please note the return type: another reader! Can you see the power of composing abstractions instead of evaluating them?

That function is a program composed by another two programs in itself. We don't need to "evaluate" the result of `insertArtist` for getting the ID and then calling `retrieveArtistById` with the ID, thing that you _normally_ do imperatively. In our case, we're totally _following the types_ and composing these bigger implementations based on the _primitive_ ones.

Finally, you need to know that your algebra MUST reflect the _published contract_ for the domain (or part of the domain) that you are encoding there. These are your principal domain operations, you are not publishing **_auxiliary_** operations like, for example, traversing a list of domain objects for applying some transformation. That is not part of your domain, even if one of the steps of the domain operations needs to do that.

## How to interpret the algebra?

Your algebra is a contract, it is not a concrete implementation. A truly advantage of that fact is that you can create different implementations of the same contract. That is, you need to give an implementation to the operations of your domain and you can do it depending on the context.

In this case, our `ArtistService` has an `ArtistServiceInterpreter` (located in the package `io.github.alejandrome.algebra.interpreter`) that has the following implementation:

```scala
class ArtistServiceInterpreter extends ArtistService[Artist]{

  override def listArtists(): Reader[ArtistRepository, List[Artist]] = Reader {
    repo: ArtistRepository =>
      repo.query()
  }

  override def retrieveArtistById(id: Int): Reader[ArtistRepository, Option[Artist]] = Reader {
    repo: ArtistRepository =>
      repo.query(id)
  }

  override def retrieveArtistByName(name: String): Reader[ArtistRepository, Option[Artist]] = Reader {
    repo: ArtistRepository =>
      repo.query(name)
  }

  override def insertArtist(artist: Artist): Reader[ArtistRepository, Int] = Reader {
    repo: ArtistRepository =>
      repo.insert(artist)
  }

  override def updateArtist(artist: Artist): Reader[ArtistRepository, Int] = Reader {
    repo: ArtistRepository =>
      repo.update(artist)
  }

}

object ArtistServiceInterpreter extends ArtistServiceInterpreter

```

Let's take a detailed look:

1. **Implementation as a class:** we have a class that extends from our algebra, and in this case we are providing the necessary type (`Artist`) to materialize every operation that we need to implement.
2. **`override` modifier:** meaning that we are implementing the operations specified by the algebra.
3. **Interpreter materialization:** the last line of our interpreter is a way to create a _single_ instance of our interpreter for using it where needed.
4. **Implementation:** each function has its own implementation, in this case using the provided `Reader` parameters to produce an output. More of this later.

As you can see, that is a straightforward implementation.

Note that there are some operations (like `retrieveArtistById`) that have primitive types provided in advance. We can replace them and provide generic type parameters for them if we want to completely _seal_ our implementation to work with explicitely specified types.

## Repository pattern

As you noted, our application relies on persisting data in a Database. Normally we would create some DTO's and then code the persistance layer to interact with the DB. In this case, we're persisting an _entire domain entity_, instead of _mixed domains_, sometimes represented by a DTO. This pattern has a whole chapter in **_Domain-Driven Design: Tackling Complexity in the Heart of Software_** and represents one of the pilars of [DDD](https://alejandrome.github.io/ddd-essentials-1).

According to [P of EAA catalog by Martin Fowler](https://martinfowler.com/eaaCatalog/repository.html), the repository pattern:

> Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.

It is extremely important to understand that the repository (in this case the database) give us a way to persist things, but the storage model in it doesn't reflect our domain model. A co-worker of mine [said it recently](https://twitter.com/castillobgr/status/880589357173886976) and he's damn right:

> Today's mantra: the database model is not the domain model (no matter how hard ActiveRecord tries to tempt you).
> 
> — _David Castillo (@castillobgr) June 30, 2017_

The whole idea of this pattern is to give a mechanism to map between the domain and the persistence, without _messing_ with our aggregates and entities.

I'm citing (verbatim) [the advantages of this pattern](https://books.google.com.co/books?id=xColAAPGubgC&pg=PA152&lpg=PA152#v=onepage&q&f=false), explained by Eric Evans:

* They present clients with a simple model for obtaining persistent objects and managing their life cycle.
* They decouple application and domain design from persistence technology, multiple database strategies or even multiple data sources.
* They communicate design decisions about object access.
* They allow easy substitution of a dummy implementation for use in testing (typically using an in-memory collection). 

Finally, have you heard of the [Object-relational impedance mismatch](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)? Well, this is a personal opinion, but that's why ORM's are a no-no for me. In this case, you'll see how the Repo. pattern will help you to ease the pain of relying on one of those things.

It's not bad to take a look to a [very controversial opinion about it](https://blog.codinghorror.com/object-relational-mapping-is-the-vietnam-of-computer-science/).

## Implementation of a Repository

We have the following algebra defining a very simple repository:

```scala
trait Repository[A, Id] {
  def query(id: Id): Option[A]
  def insert(obj: A): Int
  def update(obj: A): Int
}
```

You can see that this is a published algebra for any repository that we want to implement. It has the very basic operations on our domain object (Artist): query by id, insert or update an entire artist. Note that the semantics of this trait are totally generic, the "Artist" part is a way more concrete.

Now let's specialize this contract a little bit:

```scala
trait ArtistRepository extends Repository[Artist, Int]{
  def query(): List[Artist]
  def query(id: Int): Option[Artist]
  def query(name: String): Option[Artist]
  def insert(obj: Artist): Int
  def update(obj: Artist): Int
}
```

As you can see, `ArtistRepository` is a more specialized algebra that implements the basic operations of a repository but also adds its own concerning operations that aren't necessarily shared with other domain objects. Having said that, you can also implement an `AlbumRepository`, `BandsRepository` and `SongsRepository` based on `Repository` and each one of them could have their own operations, with the sole condition of implementing the very basic operations published by the generic contract.

In this case, `ArtistRepository` is offering you two more ways of querying: list all the table, giving you a list of domain objects or seeking an object by the artistName. Again, note the parametrization on types.

Now, let's wonder something: _How many implementations an `ArtistRepository` could have?_

The answer is: as many as the number of RDBMS/NoSQL engines we need to provide support for!

This is another huge advantage: provide an interpretation for a repository for each possible _concrete_ database engine we need to support. With that in mind, we can swap our repository implementation easily without needing to make any kind of change to dependent layers. Even for testing, we can provide a dummy or mocked repository for our unit tests very easily.

Let's see how does it looks like our interpreter for PostgreSQL:

```scala
class ArtistPGRepository extends ArtistRepository{

  import io.github.alejandrome.config.ApplicationConfig._

  def getTransactor[T](q: ConnectionIO[T]): IOLite[T] = {
    for {
      connection     <- HikariTransactor[IOLite](driverClassName, connectionString, userName, password)
      _              <- connection.configure(hx => IOLite.primitive(hx.setAutoCommit(true)))
      databaseAccess <- q.transact(connection) ensuring connection.shutdown
    } yield databaseAccess
  }

  override def query(): List[Artist] = {
    val q = sql"SELECT * FROM ARTISTS".query[Artist].process.list
    val tx: IOLite[List[Artist]] = getTransactor(q)
    tx.unsafePerformIO
  }

  override def query(id: Int): Option[Artist] = {
    val q = sql"SELECT * FROM ARTISTS WHERE ARTISTID = $id".query[Artist].option
    val tx: IOLite[Option[Artist]] = getTransactor(q)
    tx.unsafePerformIO
  }

  override def query(name: String): Option[Artist] = {
    val q = sql"SELECT * FROM ARTISTS WHERE STAGENAME = ${name.toUpperCase()}".query[Artist].option
    val tx: IOLite[Option[Artist]] = getTransactor(q)
    tx.unsafePerformIO
  }

  override def insert(obj: Artist): Int = {
    val q = sql"INSERT INTO ARTISTS(STAGENAME, AGE) VALUES(${obj.stageName}, ${obj.age})".update.run
    val tx: IOLite[Int] = getTransactor(q)
    tx.unsafePerformIO
  }

  override def update(obj: Artist): Int = {
    val q = sql"UPDATE ARTISTS SET STAGENAME = ${obj.stageName}, AGE = ${obj.age} WHERE ARTISTID = ${obj.artistID}".update.run
    val tx: IOLite[Int] = getTransactor(q)
    tx.unsafePerformIO
  }

}
```

This class has the necessary implementation of every operation and the logic for accessing the database and retrieving the necessary data. Please ignore (for the moment) all the concrete implementation with [doobie](https://github.com/tpolecat/doobie) because it'll be another post :)

## And `Reader`?

We've seen so far three concepts: Algebras, interpreters and repositories. And I've said that `Reader` is a perfect fit for dependency injection.

Let's return to the scenario of dealing with multi-db support: how can we _inject_ the correct repository without the **[magic](https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.gif)** tricks of DI like [Guice](https://github.com/google/guice) or [Spring Framework](http://projects.spring.io/spring-framework/) (IoC)?

Maybe you're wondering why are we bypassing all these solutions that _makes our lives more easy_? Well, we'll see the functional advantages that this monad give us (in fact, I gave you one reason before: composition. Refer to the [algebra section again](#The Algebra).

Let's review our `ArtistService` again:

```scala
trait ArtistService[T] {

  def listArtists(): Reader[ArtistRepository, List[T]]
  def retrieveArtistById(id: Int): Reader[ArtistRepository, Option[T]]
  def retrieveArtistByName(name: String): Reader[ArtistRepository, Option[T]]
  def insertArtist(artist: T): Reader[ArtistRepository, Int]
  def updateArtist(artist: T): Reader[ArtistRepository, Int]
}
```

Note that the return type of every function exposed by our service is a `Reader`. This monad takes two parameters: the **_dependency_** that we want to inject and the type that the function expects to evaluate to.

You can see `Reader` as a function that goes from `A` to `B`. Something like `f: A => B`. The difference here is that the function is somewhat _lazy_. I mean: the function is encapsulated in a monadic container (`Reader`) and you need to:

1. Provide the neccesary dependency.
2. Explicitely run the function to evaluate it.

Let's take a look at our service interpreter (I'll take only one operation for the sake of brevity):

```scala
class ArtistServiceInterpreter extends ArtistService[Artist]{

  override def listArtists(): Reader[ArtistRepository, List[Artist]] = Reader {
    repo: ArtistRepository =>
      repo.query()
  }
...
}

object ArtistServiceInterpreter extends ArtistServiceInterpreter
```

Note that this function takes the dependency (our repository) and executes the database operation needed. At this point, we haven't evalued the result, for that reason we need to provide the dependency and `run` our reader.

Our microservice is based on Akka-Http, so let's see how we're invoking a service operation based on our API definition:

```scala
import io.github.alejandrome.algebra.interpreter._
def api(repository: ArtistRepository): Route =
    handleExceptions(exceptionHandlerApi){
      handleRejections(genericRejectionHandler){
        pathPrefix("api") {
          pathPrefix("artists") {
              parameter("id".as[Int]){ id =>
                (pathEndOrSingleSlash & get){
                  createHttpResponse(
                    statusCode = StatusCodes.OK,
                    entity = ArtistServiceInterpreter.retrieveArtistById(id).run(repository).asJson
                  )
                }
              }
              ... 
          }
        }
      }
  }
```

Let's see this with more detail:

1. **Repository parameter:** our API need an `ArtistRepository` to run. We'll see shortly how we're providing the implementation.
2. **Interpreters import:** the top `import` statement is bringing to the scope all the service interpreters available. In our case, we're taking advantage of our created instance of `ArtistServiceInterpreter` to call the required operation to complete this request.
3. **`.run(repository)`:** as we've said before, you need to explicitely run the computation contained into the Reader to get the value. That statement takes the dependency needed for materialize the computation. Short and sweet.

Look how easy is to call the required concrete service (interpreter) and injecting the repository for running:

`ArtistServiceInterpreter.retrieveArtistById(id).run(repository)`

Isn't that great?

Finally, let's take a look to our `Boot` class:

```scala
object Boot extends App with Api{

  implicit val system = ActorSystem("readerm")
  implicit val materializer = ActorMaterializer()
  implicit val executionContext = system.dispatcher
  implicit val logger = LoggerFactory.getLogger(this.getClass)

  val apiHost = system.settings.config.getString("akka.http.host")
  val apiPort = system.settings.config.getInt("akka.http.port")

  val akkaHttpLogger = Logging(system.eventStream, "readerm")

  val repo: ArtistRepository = ApplicationConfig.activeDatabase match {
    case "postgres" => new ArtistPGRepository
    case "h2" => new ArtistH2Repository
  }

  Http().bindAndHandle(handler = api(repo), interface = apiHost, port = apiPort) map {
    binding =>
      akkaHttpLogger.info(s"Readerm API Bound to address ${binding.localAddress}")
  } recover{
    case ex =>
      akkaHttpLogger.error(ex, "Failed to bind Readerm API to {}:{}", apiHost, apiPort)
  }

}
```

And our configuration file:

```hocon
postgres{
  driverClassName = "org.postgresql.Driver"
  connectionString = "jdbc:postgresql:postgres"
  userName = "postgres"
  password = ""
}

h2 {
  # Pending implementation
}

activeDatabase = "postgres"
```

In this case, we're creating the neccesary database repository implementation upon starting our application based on a configuration that states what database engine we're going to use. The materialization is here:

```scala
val repo: ArtistRepository = ApplicationConfig.activeDatabase match {
    case "postgres" => new ArtistPGRepository
    case "h2" => new ArtistH2Repository
}
```

With that we're ensuring the creation of the correct dependency and its further injection.

## Conclusions

It's a long post, I know, but I wanted to illustrate as detailed as possible these patterns and its implementation. If you have time, please take a look at the three further sections of this post in which I detail some considerations about this implementation.

So far, we've covered:

* The algebra and interpreters and its benefits from a functional point of view.
* The repository pattern, its advantages and an example implementation.
* Reader monad for dependency injection and how we can express computations based on its monadic properties.
* The combination of all this topics in a working implementation.

Remember to [take a look at the project](https://github.com/AlejandroME/ReaderM).

This approach succeeded in our real-world use case and enabled us to understand how these functional capabilities can make our software more expressive and maintainable.


## Things to note

When building this project, I faced a couple of issues that I want to document here, in case that you encounter it:

* When you run a `Reader`, it evaluates to `Id[T]` instead of `T`. That was strange for me because I've used `Kleisli` before and with that abstraction the same operation (`run`) gave me a `T`. I asked in Stackoverflow and [here is the answer for that behaviour](https://stackoverflow.com/a/44872725/478030).
* When I was learning how to use `doobie` I faced a behaviour in where IntelliJ wasn't capable of recognize the `.query` combinator. I lost a lot of time double-checking if I done something wrong and I came across with [this](https://github.com/tpolecat/doobie/issues/206) and [this](https://youtrack.jetbrains.com/issue/SCL-10091). So, the conclusion is that there's a bug with IntelliJ and the `sql` interpolator. If you see that there are errors in `ArtistPGRepository` when using IntelliJ, don't worry, it's an unsolved bug of the IDE, and not of the project.

## Naivety of this first implementation

I'm thinking about iterate this project with new improvements and to use it as a sandbox in where I can test new libraries / patterns. So I'm deciding to use it as the codebase for future posts here.

There are some _issues_ at this time that I have to solve:

* Error handling.
* Implementation of the rest of entities of my domain.
* General refactors.
* Add more unit tests.
* Improve the doobie's Transactor implementation.
* Automate schema creation and data population.

So, you'll see some things that could make you think about the naivety of the implementation, but it is the first version.

## What do we do next?

I had a great time playing around with doobie and I think that it deserves a separate blogpost.

Also I want to improve the model adding more complex operations to the algebra for practically demonstrate the compositionality that one can get of this approach.

For now, I will have my next post about doobie as soon as possible.

Thanks for reading! :)