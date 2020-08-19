---
layout: post
title:  "Notes on 'Working in Public'"
date:   2020-08-18 12:00:38 -0400
authors: Sid Shanker
categories: books
---

I just finished and really enjoyed reading [Nadia Eghbal's](https://twitter.com/nayafia?lang=en) new book,
[Working in Public](https://press.stripe.com/#working-in-public). It's a fantastic analysis of the current
landscape of Open Source Software, and digs into the primary problems facing the OSS world today.
Some of the questions she sets out to answer are:

* What is the *actual* work of an Open Source maintainer?
* What are implications of Github's dominance in the OSS world?
* What is the right mental model to understand the economics of open source?
* What are the current and potential future models for funding open source?

The book, in addition to posing these questions, is full of insight--if you are interested at all
in the economics of how open source software production works, or more generally in content
production online, I'd highly recommend picking it up! (and like other Stripe Press books, it is
beautifully printed, so get the physical copy!)

Here are some of my big takeaways:

# The Mythology of Open Source vs. The Reality

Eghbal begins the book discussing some of the history
of Open Source, and some of the original evangelists of
the movement. A lot of the early ideas about open source match
up with the current mythology -- that open source is all about
*lots* of developers collaborating together to build software
together. This is the premise of Eric S Raymond's "The Cathedral
and the Bazaar" -- that with lots of developers looking at a project,
software will fundamentally be developed differently to closed source
software. For instance,
it was in this kind of thinking that the idea that "all bugs will be shallow"
with lots of eyes was developed.

Eghbal points out, with data, that for most projects, it is in fact a very
small number of contributors doing the bulk of the work. Rather than being
a utopia where any number of developers can chip in their free time, developing
software often *looks* a lot more like the development of proprietary software,
with the exception of the financial reward that usually comes along with that
type of development, and with the additional burden of having the world have
access to your issue manager!

# The Implications of Github's Dominance

An exploration that I found really interesting was Eghbal's discussion
of what the implications of Github being so dominant in the open source world
are. An analogy that she uses is that Github is like a piece of physical infrastructure,
like a highway. By having a standard interface, contributing to a new project on Github
is far easier than contributing to a project would have been in the world of mailing
lists.

The downside, shared by other large internet platform, is that it is harder for creators
to experiment with different approaches to managing their projects, or with different
approaches to making money, as some of these changes would require making changes to
the platform itself. While Github continues to build features to help developers, many
of the ideas Eghbal lists towards the end of the book for creators to make money couldn't
be implemented without Github implementing *something*.

# $$$

A large portion of the book is dedicated to understanding the costs of production
in OSS. One of the key insights here is the mental model Eghbal uses to understand
what the problems are with the cost of production. She argues that unlike with other
products in the world, "managing open source code requires separating its production from
consumption: treating them as not one but two types of economic goods".

Open source code in "static" state itself is a public good, and does not suffer from
tragedy of the commons. Many people can consume, say a Javascript library, and that on
its own does not make the Javascript library worse.

Production of open source software, on the other hand, does suffer from a tragedy
of the commons. Only a small number of members of a project are capable of contributing
typically, yet any user of the software can raise an issue, and therefore consume
the attention of maintainers, which is finite.

This becomes a helpful framework for when Eghbal then begins to discuss how to think about
how OSS *is* and *ought* to be funded. For instance, she would argue that in general, charging
for software itslef does not make any sense, since it is not the good that is being used up, and rather
efforts should be expended on figuring out how to manage the *attention of maintainers*.

# Conclusion

It's easy to forget when you've been immersed in software development that OSS really
is strange. *Working in Public* does a great job of providing some models to think about
how OSS functions as a form of economic production, as well as a better understanding
of what OSS work actually looks like.
