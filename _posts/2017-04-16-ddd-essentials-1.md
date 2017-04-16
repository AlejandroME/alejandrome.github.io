---
layout: post
title: "Domain-Driven Design Distilled, part I: Essentials"
date: 2017-04-16 13:05:00-0500
categories: ["theory", "domain driven design", "ddd", "vaugh vernon", "bounded context", "ubiquitous language"]
---

One of many technical debts that I have with myself it's DDD, which came to light one day of 2013 when my boss at that time just entered the office waving a copy of the [_DDD Bible_](http://a.co/5MYX4dF) saying that it was a recomendation of a friend. Since then I tried to read the whole book two or three times, leaving it because it is dense and because I didn't found enough self-explainatory examples for the knowledge that I had of the topic at that time.

Last year, [Yuji Kiriki](https://twitter.com/ykiriki) finished collaboration for the amazing [_Domain-Driven Design Distilled_](http://a.co/5251Z9Y) by [Vaughn Vernon](https://twitter.com/VaughnVernon). Since then the book was on my to-do list because it would be a more easy approach to DDD and I wasn't wrong about that: if you don't have any practical experience with this concept, if you find current resources about the topic overwhelming or even you don't know what DDD is at all this book is a _must-read_.

In this post I will share some of the basic concepts of this subject, based on Vernon's book and hopefully you can explore more on your own.

Being this a somewhat theoric and dense topic this will be one of many (I don't know how many of them) posts dissecting Vernon's book. Hopefully I'll be doing the same job with the next two that I am going to read: [Implementing Domain-Driven Design](http://a.co/d3TH6fy) and (again) [Domain-Driven Design: Tackling Complexity in the Heart of Software](http://a.co/5MYX4dF).

Also, expect to have implementation examples. And in this case I'll help myself out with the excellent [Functional and Reactive Domain Modeling](http://a.co/cELnHhU) by [Debasish Ghosh](https://twitter.com/debasishg).


## But, what is DDD?

Domain-Driven Design was a term coined by Eric Evans in his book **_Domain-Driven Design: Tackling Complexity in the Heart of Software_** (referenced at the beginning of this post) in which, in a few words, encourages you to design your software as a model of your problem space (the business process, **_not related to BPM or something like that_**) working closely with the people in charge of the _modus operandi_ of the company. In the book these people are called the _domain experts_ and the whole idea is to design a model in which you (as a developer) and the domain experts can communicate and reason about it, refine it and evolve it.

DDD is not a framework (think Scrum, for example) but a set of tools that helps you to design and implement _good software_.

At the end, citing Vernon: _The software is the model and the model is the software_. That's the main goal.

This is the very basic meaning of the term but generally speaking it covers a set of definitions, patterns and practices that I will discuss in this and subsequent posts.

Another good definition for this term can be found on this [Stackoverflow question](http://stackoverflow.com/a/1222488).

## Strategically and Tactically designed software

When I just finished the book and was making up my mind with notes and highlights about the text I found two concepts that caught my attention: **Strategic design** and **Tactical design**. In fact, all of the book's chapters treats their topic based on the premise that it is a SD approach or a TD one.

Citing book's definition about these concepts:

> The DDD strategic development tools help you and your team make the compititively best software design choices and integration decisions for your business.

and

> The DDD tactical development tools can help you and your team design useful software that accurately models the business's unique operations.

As you can see, when you are designing your application based on a domain approach you have to be careful about aligning the business needs with decisions taken based on your scope. I know that this sounds too obvious but unfortunately in the world of enterprise software that's not true, specially when you see the implementation of their critical systems.

The idea of the strategic approach is not free: you have to carefully craft your domain and understand what is the business about, separating the technical boundary (at this stage of the process) from the definition because you will do the exercise of modelling with domain experts that don't have any idea about technical concepts. I will expand my reasoning about this later on this post.

Now, the tactical approach is condensed in this two questions: How will I reflect that strategic decisions/approaches/architecture defined, in code? and how will I guarantee that my code reflects closely this model?

Don't take me wrong: if these things still sounds too obvious for you and you had the opportunity of working in an application with nonexistent design you'll be familiar with the next concept.

Summing up, this two concepts forms the base of what DDD intents to do: **_Effective design_**. That is, stategic design with tactic design implementation.

## _"Big ball of mud"_

Quoting _another quote_ of the book:

> Questions about whether design is necessary or affordable are quite beside the point: design is inevitable. The alternative to good design **is bad design, not no design at all**.

and

> **(...) if you have five software developers working on the project, no design will actually produce an amalgamation of _five different designs in one_**.

When the boundaries between concepts dissapears, when _something-calls-someone else-and someone else calls another one-_, when everything in the software is deeply tangled and a change impacts others... that's a big ball of mud. That situation, unfortunatelly is the common sight in our industry, specially in complex and critical enterprise systems.

The previous quote is an excellent indicator of the fact that there is _no design_ but _bad design_.

Makes sense now?

## Speaking one and only language

One of the main concepts of DDD is the proposal of an _universal_ language that everyone involved in the design phase can understand, no matter if the person is a developer or not. Also, that language distinguishes the meaning of the same term in different contexts, for example, when you are talking about the concept of a **Policy** in the insurance world, that could have a different meaning in different stages of the business process: claims, issuing, underwriting and so on. 

I just have mentioned two base concepts of DDD: _Bounded Context_ and _Ubiquitous Language_.

### Bounded Context

A bounded context is a boundary that delimites the context of specific business terms and their meaning. Citing Vernon's book:

> (...) a bounded context is a semantic contextual boundary. This means that within the boundary each component of the software model has a specific meaning and does specific things.

Returning to the example of the insurance policy concept: knowing that a _Policy_ has a different context depending on the process phase makes a clear point about how to model this concept in software.

Based on my personal experience, the organizations with whom I have worked with tries to _summarize_ a particularized concept like this in a _global_ way, trying to make it **Canonical**. I lost the faith in that word because what [canonical models](http://www.enterpriseintegrationpatterns.com/patterns/messaging/CanonicalDataModel.html) preaches is very disconnected from what the real implementation is or what in reality that is. In this book this problem is treated as part of the Big Ball of Mud, because we tend to have, in this example, an unique Policy with all its attributes during its lifetime, meaning that you'll have a greasy and complex model that is shared partly between each execution step. It wont be easier to maintain and evolve.

That said, drawing this boundaries helps you to isolate specific business behaviours depending on the context and also makes a statement in which is the scope of the concept based on this context.

For making this point crystal clear:

> The software model inside the context boundary reflects a language that is **developed by the team working in the bounded context and is spoken by every member of the team that creates the software model that functions with that bounded context.**

Also note that I have only spoken about a single Bounded Context. What about if our problem has many of them? How do they communicate and correlate? Well, I'm not going into further detail in this post but you can take a look [here](https://www.infoq.com/articles/ddd-contextmapping) and [here](https://es.slideshare.net/StijnVolders/context-mapping). 

And now, when you delimite these concepts within their respective boundaries you are forming a general language, that is the _Ubiquitous Language_.

### Ubiquitous Language

Normally your problem domain isn't limited only to one bounded context ---If that happens, DDD is not exactly what you are looking for--- and while working to define all this boundaries and their connections you are forming an Ubiquitous Language, that is a generalized language that everyone understands, no matter their role in the team. Usually (in its visual form) is modelled with _basic UML_ but this is not a constraint. The written form, meanwhile, has to be code, remember: **_In the end, the model is the code and the code is the model_**.

That implies:

* **Naming things:** it is common to see that we (as developers) are challenged when it comes to name things, and in the context of DDD, naming things matters, and matters a lot. This is not a "rule" from DDD, but it is reinforced there: the code must be self-explainatory. That is, if someone new has to read it this person could understand the business through the code.
* **UL is the blueprint of everything:** every change in the model has to be reflected through the Ubiquitous Language and no _obscure, undocumented logic_ has to be done to achieve a business constraint. Even worse, if you detach your code implementation from the UL you're not speaking one and only language anymore, thus you are throwing all the effort that you and your team have invested, away.
* **Documents aren't trustworthy:** If your references for understanding some business concept are formal documents there is something wrong. This point is connected to the last one emphasizing about the Ubiquitous Language as the unique source. If you are looking for an external resource then there is something not explicitly stated in your UL. Make it explicit in the model.

## What you can conclude from all this?

I have summarized the concept of _Bounded Context_ and _Ubiquitous Language_ in a short way to illustrate what it is all about. Also, we have seen how DDD can help us to define and place relevant concepts of the addressed problem in relevant contexts (seggregation). I reviewed a _Big Ball of Mud_, ample treated throughout the book and how DDD can help us to not fall into that.

Finally, we've seen the approach that Vernon gave to the book, looking through a Strategic and Tactic vision of all DDD lifecycle.

This post is the start of a new series in which I will review the basic concepts that forms DDD and how can you use it when designing and developing software.

Last, but not least, I know that this is a complex topic, so I will leave some resources at the end of this post for compliment the lecture. This, as the previous post, is written in a rather informal way, trying to summarize what I've read from this book and the previous one. If you find some imprecision or mistake, please let me know in the comments!

## Resources

* [BoundedContext. Martin Fowler](https://martinfowler.com/bliki/BoundedContext.html)
* [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html)
* [Model Integrity Map. Wikimedia](https://upload.wikimedia.org/wikipedia/commons/7/73/Maintaining_Model_Integrity.png)