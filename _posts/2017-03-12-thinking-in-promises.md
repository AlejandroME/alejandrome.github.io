---
layout: post
title: "What I understand about 'Thinking in Promises'"
date: 2017-03-12 13:05:00-0500
categories: ["theory", "promises", "thinking in promises", "mark burgess"]
---

I just ended reading [_Thinking in Promises: Designing Systems for Cooperation_](https://goo.gl/JUE0YQ) by [Mark Burgess](https://twitter.com/markburgess_osl) and I want to write about it because I think that it is a good exercise for trying to grasp some definitions treated throughout the book that seems a kind of abstract.

This book was a recommendation of [Juan Pablo Vergara](https://twitter.com/juanpavergara) who spoke briefly about it during our [Scala meetup](https://www.meetup.com/ScalaMDE/) session in december 2016. Also I have seen mentions of this book by Jonas Bóner in his talk [_bla bla microservices bla bla_](http://jonasboner.com/bla-bla-microservices-bla-bla/) that looks interesting seen by the point of view of microservices and distributed systems.

## Promises as _Contracts_

When I saw the term **promise** the first thing that came to my mind (inevitably) was the concept associated to Futures and Promises, as seen in [Scala](http://docs.scala-lang.org/overviews/core/futures.html) and [Javascript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Mark states that our world actually works in an _impositional_ way, meaning that most of the actions that rules everything are impositions rather than _promises_ and he uses a clear example of this:

> **Instructions for cleaning a public restroom:**
> 
> * Wash the floor with agent X.
> * Mop and brush the bowls.
> * Refill soap.
> * Do this every hour, on the hour.

The outcome of this task is not defined, instead you have instructions that you **must** follow and the _assessment_  of the result could be different, depending on the _agent_ that evaluates it.

Changing that set of tasks by a promise approach looks like this:

> * I promise that the floor will be clean and dry after hourly checks.
> * I promise that the bowls will be clean and empty after hourly checks.
> * I promise that there will be clean towels in the dispenser after hourly checks.
> * I promise that there will be soap in the dispenser after hourly checks.

What can you see? Is the first approach clearer than this or is the other way around?

Note that in the promise formulation all the statements has a clearer and _measurable_ outcome. Also, when you have a defined vision of what you want that makes you _predictable_ and the results of your promises are _reliable_, _repeatable_ and _available_.

From the book:

> Promise theory is not a management ideology; it is an engineering framework for coping with uncertainty in information systems.

and:

> It is a bottom-up constructionist view of the world

Makes sense?

But, why am I talking about contracts? Well, from what I understood throughout the book, thinking in promises is not only about draft statements _à la promise way_ but reengineering how systems work in a _cooperatively_ way. 

One of the things that I can keep comparing is the microservices approach, and more generally, the approach of some service that offers you (as a consumer) some capabilities that, as you would expect, produces predictable outcomes ([hello, FP concepts!](https://en.wikipedia.org/wiki/Functional_programming)). For example, if there is a microservice that offers you an API for managing a customer database (CRUD operations), you would expect _a contract_, that is, what is the structure that you have to give to the API for performing one of these operations, what is the structure that the service will use for answer your petition And much better: how do you describe this workflow as promise statements? Try to think how can you do this.

That example is confirmed by the book, according to this:

> SOA is an example of a promise-oriented model, based on web services and APIs, because it defines autonomous services (agents) with interfaces (APIs), each of which keeps well-documented promises.

And a disclaimer here: **SOA is NOT any of these [vendor](http://www.oracle.com/us/products/middleware/soa/overview/soa-overview-1678943.html) [approaches](https://www-01.ibm.com/software/solutions/soa/)**

## Definitions

Because I am trying to reinforce the concepts treated in the book, I will mention some of them and also I am going to ask some questions for further debate.

### From the book:

### What is a promise?

> (It is) a kind of atom for intent that, when combined, could represent a maintanable policy.
> 
> It is a publicly declared intention to an audience (scope).

### What is an intention?

> It is a subject of a possible outcome.

### What is an imposition?

> It is an attempt to induce cooperation in another agent (hints, advice, suggestions, request).

### What is an obligation?

> It is an imposition that implies a cost or penalty for non-compliance.

### What is assesment?

> It is a decision about whether a promise has been kept or not.

Why I have cited these definitions? because, if you haven't read the book yet it is important to know these concepts beforehand because they are the basis of the theory.

Let's focus on the assessment part.

## _The truth is in the eye of the beholder_

When you, as an agent, promise something to someone, that external entity will assess the outcome of your work. The thing that I find amazing is how an assessment could be entitled to the perspective of the beholder, even in information systems (that tends to measure everything as accurately as possible). What you see as correct other entity can see it as wrong and that's good. 

Burgess states that **the beauty is in the eye of the beholder** because there's _cultural_ norms when judging the value of something.

Depending on the entity of the system that assess something some outcome could be right or could be wrong, and that is a perspective hard to achive or to think of as a software developers because we tend to measure everything by the same yardstick, no matter the context.

Sounds trivial but that catchphrase is very very powerful because obligues us to think about the multiple perspectives, contexts, responsibilities and relationships between the agents of our system. That changes everything because that state of mind is unreachable with the _standard_ **imperative reasoning** (_if this, then that, no matter what)_.

## How do promises and agents compose and interact?

Based on what Mr. Burgess presents during the entire book, I draw parallels between the [actor model](http://www.brianstorti.com/the-actor-model/) and the concept of promises. When you, as an individual agent promises to do something and some observer or consumer assess that work, you are:

* Segregating responsibilities through different actors.
* Delegating _granular_ tasks to agents.
* Composing outcomes for complex and long-running tasks.

Am I wrong when I say that I find a relationship between what the actor model proposes and what promise theory tries to achieve?

From the book:

> Promises fit naturally with the idea of services. Anyone who has worked in a service or support roles will know that what you do is not the best guide to understanding: _"Don't tell me what you are doing, tell me what you are trying to achieve!"_

Again, the interface for an actor is basically what kind of tasks are delegated to it and what the actor will do, seeing it as a piece of a greater task. That last part is somewhat validated according to this:

> The goal of Promise Theory is to take change (dynamics) and intent (semantics) and combine these into a simple engineering methodology that recognizes the limitations of working with incomplete information. Who has access to what information?

## "A Collective Design"

A thing that I have found very interesting is the way in which every agent has a role and how that roles forms what is said to be a _collective superagent_. The first part of the book (the first two chapters) treats promises as a single unit but the rest of the book focuses on how that entities have different properties that can take different roles in the system.

I see parallels with the actor model (again) and distributed systems about the consensus that every agent promises and the roles (by appointment, by association and by cooperation) that they takes.

Also, a thing that I cannot completely understand is the [Nash Equilibrium](https://en.wikipedia.org/wiki/Nash_equilibrium) for consensus. It is interesting because, being a concept that fits into [Game Theory](https://en.wikipedia.org/wiki/Game_theory) it sounds more like a competition for an "optimal" solution of a game rather than an agreement. At this point I am trying to figure out if something like consensus algorithms (like Paxos) fits somehow in this idea.

For illustrate this concept, the book offers an example about agreement:

> Suppose four agents: A, B, C and D; need to try to come to a decision about when to meet for dinner. If we thunk in terms of command sequences, the following might happen:
> 
> 1. A suggests Wednesday to B, C and D.
> 2. In private, D and B agree on Tuesday.
> 3. D and C then agree that Thrusday is better, also in private.
> 4. A talks to B and C, but cannot reach D to determine which conclusion was reached.

Sounds simple (and the book says that is a _classic distributed computing problem_) but I understand it only at some extent. What is remarkable is the vision that the promise concept brings to the distributed systems design and I say that because of this quote:

> The lesson (of the stated problem) is that, in a world of incomplete information, trust can help you or harm you, but it's a gamble. The challenge in information science is to determine agreement based only on information that an agent can promise itself.

Sounds like the CAP Theorem.

## Continuous Delivery

One of the things that caught my attention the most was the mention of CD and DevOps in various parts of the book and the analogy of a _continuous_ system with _repeatable_ outcomes. Things like _containers_, _versioning_ and _branching_, for example, are related specifically to software development but the author brings that to the conversation as a way to think about the semantics and dynamics that each of them represents in a system in which this traits makes the results predictable.

## Final thoughts

This post is more like a personal abstraction of the core concepts that I understand about the book. There is more of them that I didn't mentioned here, but it is a start because I want to make my mind with the main ones and then make another post with my findings.

If you see a _misconception_ about things that I have said here, please leave your comments.

Thanks for reading!