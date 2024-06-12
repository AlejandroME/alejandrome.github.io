+++
title = "Understanding Software Dynamics Book: Prelude"
date = "2024-06-12"
description = "Or participating in a book club"
summary = "An intro to the Understanding Software Dynamics Book, chapters 1 and 2"
readTime = true
math = true
toc = true
tags = ["understanding software dynamics", "performance", "book club"]
showTags = false
hideBackToTop = false
+++
Phil Eaton (one of my influences) started a [mailing list-like book club](https://eatonphil.com/2024-understanding-software-dynamics.html) in late May for the [Understanding Software Dynamics](https://a.co/d/glF9G02) book. At a glance, the book delves into a daunting topic for someone who haven't been this deep into systems performance, but I have to say that it's an interesting opportunity for multiple things:

- Reading about a "novel" topic, and trying to _digest_ what I understood about it.
- Read what other people have to say about the topic, and learn from them.
- Get the encouragement to go deeper.

The book club will kick off the discussion for two chapters at a time each week, and I am aiming to put my somewhat messy notes here, along with the _open questions_ that I have. With this format, I hope that I can come back and answer some of those, based on my learnings.

Having said that, let's go!

## Chapter 1: My program is too slow

The takeaways from my reading of chapter 1 are captured in the following ideas:

1. What is too slow?
2. General terminology
3. Tail latency
4. Five fundamental resources

### What is too slow?

In Chapter 1, Richard starts talking about two types of programmers:

- Those who can talk about — with numbers — about their program and why it is not behaving as expected.
- Those who can say: _"My program is slow, and I don't know how to make it fast"_.

I am one of those programmers in the second type. It's interesting to observe how working at higher abstraction layers _limits_ the scope of possibilities about what could go wrong with a program. When you're writing _enterprise software_ with _known tools or frameworks_ (say, Java with Spring Boot), you're thinking about a library that may be causing an undesired behaviour, or a misconfiguration in your deployment, a missing flag in the JVM running your program, and so on. You rarely think about _a cache miss_ (unless it's a cache server 😅), or a SIMD computation. That's my observation after spending too many years in the consultancy world.

As for the first type of programmers, they know how their program should behave, most of the time by using [standard numbers every programmer should know](https://static.googleusercontent.com/media/sre.google/en//static/pdf/rule-of-thumb-latency-numbers-letter.pdf) ([you can see a nice visualization for those numbers here](https://colin-scott.github.io/personal_website/research/interactive_latency.html)), also mentioned as **order of magnitude estimations**.
Knowing that these numbers are approximations, the author says that the true learning is here, since your estimation may be wrong, or right, depending on the heuristics you use. Still, understanding the desired vs current behaviour, and running some back-of-the-envelope calculations to set your expectations greatly helps to _level the ground_, and to communicate your expectations based on numbers, not in guesses.

Just to wrap up this idea: knowing the load that your service will take, the interactions of the service with other services — a _service_ is an umbrella term here — and the runtime of your program, you can estimate what would be the time your program should take, and also help you our rule out scenarios that may not fit such estimation. Quoting the book:

> Knowing the estimates in Table 1.1 will also guide you in identifying the likely source of a performance bug. If some program fragment takes 100 msec more than you expect, the problem is unlikely to be related to branch misprediction, whose effect is 10,000,000 times smaller than 100 msec. It is more likely related to disk or network times, or as we will see in later chapters, to long lock-holding times or to waiting on long RPC sub-requests.
> 
> We will design and build observation tools and data displays. As you use them, get in the habit of predicting within an order of magnitude what you expect to see. Once you are practiced at this prediction-observation-comparison loop, you will rapidly start spotting the odd stuff.

This is also referred to as the **thought framework** in the book: _"with the offered load, and a system under stress, what are the observations we can capture, how do we reason about what we see, and then how do we change those observations?"_.

### General terminology

There is a lot of terms that I thought I knew, but most of the time I took them for granted. It was great to see these definitions explained in a holistic way, and not tied to buzzwords, or the _trending design pattern_ (microservices, anyone?). Some of the terms that caught my eye were:

- **Offered load:** the _advertised_ capacity of a service. May be measured in number of requests (or transactions) per time unit (e.g. 100K reqs/minute).
- **Service:** a collection of programs responsible for handling a transaction.
- **Tail latency (explained below)**
- **Execution skew:** the variation (in time units) of the completion time of a request.

### Tail latency

It would be futile to try and explain again [what tail latency is](https://brooker.co.za/blog/2021/04/19/latency), however, I do think that the p99 discussion mentioned in the book was an _eye-opener_ for me, mostly because of the true impact of this latency when you're able to reduce it by 1x, 2x, and more.

### Five fundamental resources

Those being: CPU, memory, disk, network, and if your program is comprised of a bunch of threads, then the critical section of the program would be the fifth one. The interesting part about these resources is to see the amount of heuristics that rule each of them, and how those influence the measurement process. If you forget a variable, that mistake may yield flawed measurements.

Learning how to observe these factors is one of the author's goals in the book.

## Chapter 2

This chapter was _denser_, since we started to talk about CPU measurement, with an example. To follow this along, a _"basic"_ knowledge of the CPU is fundamental (notice the quotes). My comments here are scarce, other than a bunch of _unanswered questions_ that I want to answer first (outlined in the open questions section).

So far, things that I liked about the chapter are:

- Richard followed the **thought framework** while running the experiment. Starting from an _odd scenario_ (a program that measures the performance of the `add` instruction), and with an expectation set, he tries out different scenarios until he finds the optimal approach.
- He delves into the eternal _time-measuring dilemma_.
- The historical introduction — and refresher — to the CPUs evolution across time.


## Some learnings from the book club

- That there are [_latency charts_](https://www.agner.org/optimize/instruction_tables.pdf) flying around there.
- [This version](https://github.com/sirupsen/napkin-math) of Jeff Dean's numbers every programmer should know.
- That there's a [layman's version of the Patterson book](https://archive.org/details/inside-the-machine-an-illustrated-introduction-to-microprocessors-and-computer-architecture-pdfdrive/mode/2up) to get acquainted with Hardware Architecture (which I'll probably read)

## Open questions

Chapter two has an interesting disclaimer:

> If you are not very familiar with the terms instruction fetch, pipelining, cache memory, and (for the next chapter) virtual memory, now would be a good time to review a computer architecture textbook, such as [Hennessy and Patterson](https://a.co/d/7jsoTUo).

Of course, most of these concepts flew over my head, and my specific open questions are:

- How does the instruction fetch process work?
- How does pipelining work?
- How does L1/2/3 cache work?
- Refresher on Memory (virtual memory, pages)
- Out-of-order execution

## Closing remarks

This post (and subsequent ones in the series) aims to be my messy notes, hopefully to orient me to learn and fill in the gaps. If someone besides me is reading this, thanks!

I hope this may be useful in the future