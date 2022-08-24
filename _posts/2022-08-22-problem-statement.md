---
layout: post
title:  "The Power of a Good Problem Statement"
date:   2022-08-22 00:00:38 -0400
authors: Sid Shanker
categories: learning
---

At Twitter, I read and wrote a lot of technical design documents – documents that describe a high-level approach to solving a particular problem. One of the key determining factors as to whether a doc was effective or not was how well written the problem statement in the document is.

It’s very typical to have a problem statement at the top of the doc – the goal here is typically to flesh out the why behind your document. If you’re proposing a change to a system, this is where you flesh out the motivation for that change.

It’s crucial to have this fleshed out well for a few reasons:

## You want to get your teammates bought into whatever it is you are trying to do

A major part of sharing design docs is to persuade and get buy-in from other teammates on a particular approach. You’ve likely written a doc because you’re hoping to communicate to others what you intend to do, and probably because you’re also hoping to get feedback. In addition to persuading your teammates of a particular approach, it’s also important to convince people that the project is worthwhile to do in the first place.

It’s hard to provide feedback on a technical design if you’re unclear on what the author of the design was intending to solve. Ambiguity around what is actually being solved can lead to avoidable arguments about approach. Furthermore, it’s hard for leadership to get excited about a project if they are unclear about what is being solved.  

## It’s easier for readers to be engaged

At a more basic level, your teammates are very busy! Don’t make them do the extra work of reading between the lines to figure out what you are trying to do. It’s also easier to stay engaged and alert reading a document if the author has outlined a juicy problem! Saying “the Order class is factored poorly” might not be very motivating, while saying, “Our developers take at least two weeks to even make basic changes to the order flow” might pique their interest!

## It helps ground you as you scope your project

One of the trickiest things about project design is establishing clear success criteria. When is the project actually complete? Lack of clear success criteria is a major cause of scope creep in projects. While a good problem statement may not solve this problem completely, it at least gives you the ability to come up with an initial set of success criteria – here’s a problem, we have succeeded in the project when this problem is solved. The original problem statement can be a good guide to whether or not you should take on additional work. Is this feature important to solving the problem that we set out to solve? If not, then maybe we cut it from the scope.

Alright – so if I’ve convinced you of the importance of having written a good problem statement, what are some of the key elements of a good problem statement?

## It describes the problem in a way that relates to business objectives

It’s really important when writing a problem statement that you relate the problem being solved to something the business actually cares about. Something developers particularly struggle with is thinking that problems that they encounter ought to be apparent to everyone else. Here’s an example:

“The OrderFlow code path sees hundreds of exceptions every minute”

This is not an effective problem statement, because “Exceptions”, on their own, are not likely something that matters to the business. Does this actually affect end-users? Does it waste developer time because of spurious pages? Let the reader know! This could be written better as:

“2% of users placing an order see an error, causing them to give up, resulting in a loss of $X every month”

Or

“We waste 20 developer-hours a month dealing with pages related to the OrderFlow code path”

In these problem statements, we describe the problem in terms of the implication of those exceptions, and put them in terms of something that the business cares about. The business likely cares a lot about making money, as well as ensuring their expensive developers are as productive as possible. With these descriptions, it’s much easier to get leadership to care about your project, as well as engage your colleagues.

## Be quantitative

This is alluded to in the previous point, but being quantitative in how you describe a problem can really help sell your teammates that your project is worth pursuing, and makes prioritization decisions much easier.

Instead of “Developers waste time because changing Foo is complicated and error-prone”, try to quantify the amount of time developers waste, or the cost of the bugs you are alluding to.

I think one of the reasons that developers are resistant to doing this (and why I have sometimes been resistant to this in the past) is because they are afraid of putting nonsense numbers in writing, and getting actual numbers is really difficult.

If you feel this way, just remember the reason you are doing this. This is an exercise in problem-sizing and prioritization. Actual precision isn’t important. Honest guesswork is likely good enough.

## Does not presuppose any particular solution

Another common pitfall I’ve seen is developers actually not writing a problem statement, but just describing whatever they are trying to build. For instance, “Currently, the company does not have a way of measuring the latency of certain code-paths”. The problem with this problem statement is that it does not describe a problem, it is describing a solution “having a way of measuring latency”.

Don’t assume that your readers will think that having a means of measuring latency is important, describe what the problem is with not measuring latency!

“The company’s churn rate is 20% higher than the industry average. Based on qualitative evidence, we suspect high latency could be a cause, but we cannot know because we have no means of measuring latency”.

In this problem statement, we reframe it away from the “solution”, and instead towards the actual problem and motivation for needing a way to measure latency

# Conclusion

I hope these suggestions help you write better design docs!