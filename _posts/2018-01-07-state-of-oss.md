---
layout: post
title: OSS Importance and how to defeat the fear of contributing
date: 2018-01-07 15:05:00-0500
categories: ["oss", "open source", "thougts", "community"]
---

Before I start this post, I have to say that committing to a blog is a hard thing to do, because you have to sort all your daily life responsibilities (work, personal life, leisure) and find a balance between those activities and other things like reading a technical book, writing a blog post, contributing to some project and so on.
This is the first post of the year and the first post in 6 months, and, like a new year's resolution, I expect to stick to my initial goal when I started this blog: at least 1 post every month. You'll judge.

I have some ideas for next posts, some of them technical, some of them more reflexive (?) like this one. We'll see what to expect this year.

## Introduction

2017 was a very busy (but productive) year for me. I helped to our little [Scala community](https://www.meetup.com/es/ScalaMDE/) until September, month in which we've decided to take a break, hoping that in 2018 we can restart this effort and make a stronger community along the way here in Medellín. As a friendly reminder, we've recorded the last three sessions (in Spanish) if you want to [take a look](https://www.youtube.com/playlist?list=PL32KnWTdwCo1Bl3lu23m8-m4kLaabkXMh).

Also, I happily started to contribute to [cats](https://github.com/typelevel/cats), which was one of my professional goals of that year (and still is today!): submit my first PR to the community.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I&#39;m so happy to have started to contribute to an OSS project. My first PR for cats just got approved! :D <a href="https://t.co/pEGPAO1BeZ">https://t.co/pEGPAO1BeZ</a></p>&mdash; Alejandro Echeverri (@AlejandroM_E) <a href="https://twitter.com/AlejandroM_E/status/912649672354869248?ref_src=twsrc%5Etfw">September 26, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

And finally, two days before ending 2017, I started (with other mates) to translate [Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1) to Spanish. [Jorge](https://jvican.github.io/), from Scalacenter launched the effort.

<blockquote class="twitter-tweet" data-lang="es"><p lang="en" dir="ltr">Is there an Spanish speaking person interested in translating the &quot;Functional Programming course in Scala&quot;? It would be wonderful that such an important course is translated into the second most spoken native language in the world.</p>&mdash; Jorge (@jvican) <a href="https://twitter.com/jvican/status/946715071933308929?ref_src=twsrc%5Etfw">29 de diciembre de 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

We're translating and reviewing the course and, in less than a week we have more than 30% translated!

This kind of things makes me very happy and since last year I had this little idea inside my head about making, maybe, a post talking about this kind of initiatives and about why most developers don't seem to care (and why they should care) about giving a little love to the community.

Finally, I think that part of the context of this post will be in Colombia, in where it is very hard to see people doing this (they are out there, of course, but the numbers, compared to other countries, don't help much to the statistic 😕).

## Part I: "I'm not skilled enough"

I always wanted to be a committer of some project, but when you face an application/library/framework and you start to see its internals you (or at least some of us) feel like its a daunting thing that it's suited only for people with a gifted skillset or for veteran developers. This kind of myth has to be debunked because it was the barrier that blocked me from doing this before.

The thing here is that it's not only a thought that I have: this is very widespread and very distorted. Some people have this impression because universities taught us literally nothing about the basics of a language, let alone this kind of things (what is open source? why does it matters? what is our role as developers with this "movement"? how can I contribute?). Another factor that infuses this think are the same developers, our co-workers, because of their little knowledge about this or simply because it doesn't matter for them (and it's good, but the bad part is to teach its viewpoint as a universal truth).

As a side note: I fought with [impostor syndrome](https://en.wikipedia.org/wiki/Impostor_syndrome) (and I'm still doing it) and I'm scared about the high rates of this behaviour and how our employers/colleagues do little to nothing to mitigate this bunch of erratic thoughts. I mention this because, in my experience, this thing was one of the behaviour patterns that made me feel insecure and unskilled for doing something as important as adding code (real, working and good quality code) to some piece of software that I use in my everyday work.

First thing first: **Every contribution matters. EVERY SINGLE ONE**.

Read that aloud.

Every single contribution matters mean: from a typo to documentation, to translation, to clean an unused import, to improve a piece of code, to add a nuclear weapon launch function (ok, better to not joke with this). But you got the point: everything that you do for some project, no matter how insignificant it seems, matter, and matters a lot.

### Why everything matters?

Because open source software is backed with the personal effort of every one of us. A lot of very relevant OSS projects don't have any kind funding and some of its contributors work like it is their main (and paid) job. Typically, a project has a lot of areas to tackle:

* New features.
* Improve the existing codebase. This is code cleaning, adding coverage, improving performance and other tasks.
* Documentation. **This is one of the most overlooked aspects of the software development process**. Documentation, especially for an OSS project is everything, and here there are a lot of opportunities to contribute.
* Community moderation: code review, PR merging, CoC enforcement (although there are some issues with this thing, especially in Scala community) and attaching to the project goals.

It's not an easy thing to do and lots of us are benefited from that effort.

This is not paying millions of dollars upfront for a buggy product sold by a draconian three-lettered company or a red-branded company that is constantly looking for ways to make companies pay up for things. This is from the people for the people. An altruist work entirely backed by contributors.

Unfortunately, lots of us take this for granted and even some people yell at maintainers for not having X or Y feature that they need.

Now, that said, can you note the importance of contributing to this projects? A little annoying typo in the docs counts for making it easier to read to the rest of the userbase of the project.
That lacking doc about that feature that you've struggled to learn to use? Maybe it can help someone stuck with a similar problem.
Or what about that feature that you hoped to be in your favourite library? there are countless benefits of doing this.

### Method

But maybe you're wondering: How do I start? What if I don't know the internals of the library or if it is too complex? Let me recommend some easy things to get familiar with a project:

1. **Read:** the pillar of everything. Read, read, read and re-read. Read code, take notes about things that you don't understand. Read one thousand times if you don't understand at first glance.
2. **Ask:** correlated with the first one. Take your notes and contact some of the members of the project. Don't be afraid to ask, even if you think that the question is silly or you feel bad for not knowing the answer. The people in the vast majority of the communities are very friendly and open to help, especially newcomers that are willing to participate! There are different ways to get in touch: Twitter, Gitter, issues/PR's on GitHub, Slack and even IRC!
3. **Understand the development process:** every OSS community is different from the others and have different guidelines for contribution. Make sure to understand how their development cycle works before submitting a PR. This makes sure to stick to the process that the community has established. If you have any doubt, guess what? **ASK!**
4. **Get your hands dirty:** roll up your sleeves and do it! You'll see that the feedback given to your work will teach you **a lot!**

I can assure you that there's no other joy like seeing your PR approved and merged!

## Part II: "The enterprise and OSS: one-way relationship"

I think about this a lot. I see the big names with a lot of OSS projects that they maintain, which is not surprising. But seeing [a bank](https://github.com/capitalone), [a telecommunications company](https://github.com/verizon) and a couple top Scala [consulting](https://github.com/47deg) [firms](https://github.com/softwaremill) makes me wonder how ingrained are the OSS values in these companies and how they can differentiate from the rest.

Think about this: How come a bank has a philosophy of "Open-source first"? Here in Colombia, the banks are greedy entities with systems dating from many years ago which they struggle to maintain. I can't imagine telling to a developer at one of that banks: "Hey, what OSS libraries/frameworks/things do your employer support?". The answer to that would be a loud laugh or a shrug.

What about telcos? They are hated (count me in) entities that can't even figure it out how to give a decent customer service. In software, they're not better than banks and they're very distanced from the OSS reality, especially because of its nature: a lot of their software is custom-made or bought from companies specialized in that niche.

Finally, what about consulting firms? Colombia has an interesting number of them and, in the practice I see none of them doing this. Not the big and household names and not the international ones.

Why? I have a couple of theories...


### It doesn't matter

Simple as that. The companies here don't see any relevance when it comes to contributing to OSS projects or having an OSS-friendly policy (NDA's could aggravate the problem, like Amazon's case). Colombia has flourished in the last 10 years because our burgeoning software industry, but unfortunately we have the same structural problems plaguing our industry: poor university education in computer science, bureaucracy that makes impossible to have competent people in crucial positions (CTO, VP of Technology) making good decisions and the lack of vision related to technology in an organization.

We're dealing with practices dating from 20+ years ago when it comes to enterprise software development and also with the misinterpretation of things like Scrum and Agile, things that they've made a "fancy" trend (more like a circus, if you ask me).

### Developer engagement / OSS Efforts

The number of devs that are interested in embracing OSS, nowadays, is very little. If you take into account that there are people that don't invest their time (as little time as you could imagine) in things like that, because of their family/hobbies/other things, makes very difficult to see this as part of the ADN of a company.

With this, I'm not trying to say that you've to sacrifice your time with your family or your hobbies for "look" like a better developer or an altruist person. My point is that this should be a part of our everyday work (or even our free time, if you like it). But never, ever should've to be done because my company says it. If you don't like it, you're free to be apart of it.

That kind of commitment should be encouraged in some way from our work.

### Unidirectional

Why do I say that there's a one-way relationship between the enterprise and OSS? Because, basically, the enterprise made **millions** with open source technology, a thing that is not bad or illegal. The problem is, again, that they take that for granted. When I mentioned the situation in which people complain to a project because it doesn't have X or Y feature, the vast majority of that cases are people that are building something for this kind of companies. They're not willing to "lose time" contributing the feature that they need and in some cases they even build it, but for "internal use" (they not release that ever), having even more problems with that approach (you have to maintain that _fork_ of the code, with your custom feature).

## Part III: "Why should we do it?"

... because you like it and you feel identified with the effort that other people put into great software products that are freely available.
No OSS contribution should be made because other said to me to do it. It's important to understand the role of OSS in our work and in some way this post tried to list all the possible reasons, but the commitment should've made as the result of your will to do it. The industry and our work should encourage that goal to be accomplished and that's the thing that I tried to address in part II.

So, should you do it? If you feel it, go for it. If you don't, it's good, but it's also important to understand the effort that all the people voluntarily made for making great tools that you, maybe, use in your everyday work.

As a side effect, it'll impact your CV. Some companies recruits, or gives significant importance, to what you've done in OSS.

Apart from that, you'll learn a lot. Guaranteed.

## Final part: "What's next?"

I recently watched a talk by [Heather Miller](https://twitter.com/heathercmiller) about the [current OSS state](https://www.youtube.com/watch?v=WImJnCQhutc) that is worth to take a look. Also, taking a sneak peek at OSS communities helps to understand how do they work and the ideas behind the projects.

I wanted to express my opinion about this topic and encourage you if you are thinking to contribute but you don't feel capable or if you think that it's a full-time job... remember every contribution matters.

We have a long long way ahead. The important thing is that more and more developers understand the value of the open-source in our work and even in our lives and that's what I tried to do settling my point through this post.

Finally, I embarked myself in an experiment about the adoption of OSS with my co-workers. I expect at some point to share the results :)

As usual, thank you for reading!