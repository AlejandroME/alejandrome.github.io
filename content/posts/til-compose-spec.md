+++
title = "Today I Learned, vol. 1: Compose specification"
date = "2024-06-24"
summary = "Or how to live with a long standing confusion... until now"
description = "Today I learned: Compose Specification"
toc = true
readTime = true
tags = ["containers", "docker", "compose", "standards"]
showTags = false
hideBackToTop = false
+++
This is the first of a series of posts about random things that I get to learn. This one, the first, is a short story about those things you form an idea around swearing that it's true until it's not.

# What did I learn today?

That [Compose is an open specification](https://compose-spec.io/#community) from the CNCF (Cloud Native Computing Foundation). I always thought that it was Docker's 'proprietary' format to define resources in a containerized environment _á-la-kubernetes_.

[They (Docker) _partnered_ with other players to make it an open format in 2020](https://www.docker.com/blog/announcing-the-compose-specification/). I guess that it's never too late to get the news?


## Hosting services

I have an 'old' desktop computer that I use as my main server in my homelab. I host some of my services there by exposing containers with persistent volumes. Most of them are defined with the Compose specification, managed with `docker` on Linux. I try to keep everything up to date as early as it's released, but it was a long time ago since I upgraded any of the versions of my services.

This weekend, while at it, I started getting a warning:

```
WARN[0000] /myroute/to/service/docker-compose.yml: version is obsolete
``` 

And the version I had in all the compose files ~~is~~ was `3.8`.

## "The fix"

A simple one: removing the `version` line from the compose file, as this one is already deprecated.

## What does that version number mean?

[According to the docs](https://docs.docker.com/compose/intro/history/), these versions (2.x and 3.x) accounted for features introduced during the development of the specification (such as Docker Swarm features in 3.x), and it basically identifies the _file format_ and its elements. The new Compose specification doesn't seems to keep track of these changes with a version number.
