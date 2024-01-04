---
layout: post
title:  "Aids vs. Abstractions"
date:   2022-12-14 00:00:38 -0400
authors: Sid Shanker
categories: programming
---

Every year, we developers get access to better and fancier tools to do our jobs. I’m really excited – there’s a big opportunity here for us to be able to deliver software faster and with higher quality!

Something I’ve been thinking a lot about lately, though, is how we can ensure that we’re using these new developer tools in an effective way. A useful classification I’ve found for figuring this out is asking whether a tool is an abstraction, or an aid.

# How a tool can hinder, rather than help

Before I dive into the details of what these categories are, I want to talk a little bit about how reliance on a particular developer tool can actually hinder your progress.

[Line-break debuggers](https://en.wikipedia.org/wiki/Breakpoint) are tools that allow you to run a program, stop it at any point, and inspect the values of variables within it. When I first started programming, I often first approached bugs or other unexpected behavior by reaching for my debugger, and stepped through the code as a means of understanding what was going on.
	
This instinct makes sense – I wanted to quickly understand why my program wasn’t behaving the way I expected, and line-break debuggers are often an expedient means of getting there.

However, the problem here is that by immediately reaching for the debugger, I wasn’t doing the work of reading and attempting the surrounding code, and importantly, **developing a mental model for what your program is doing**. Doing the work of reading the code and understanding what’s going on is really important, since it’s a prerequisite to being able to make more substantial changes, and can give you deeper insight into the issue at hand. Debuggers are certainly a great tool for _testing a hypothesis you already have_, but are not a replacement for developing that hypothesis.

In this case, even if a debugger sometimes helped me get to an answer, I was cheating myself of the deeper understanding of my code that would allow me to make more substantial and systematic bug fixes.

# Abstractions vs. Aids

To be totally clear, in this example, I’m not arguing against the use of line-break debuggers – just that it’s important to understand how to use them to test hypotheses about the behavior of your code, not in lieu of developing hypotheses in the first place.

I’ve been thinking a lot recently about this idea that adding a new developer tool could be a hindrance, especially in the light of the release of AI-driven tools like Github Copilot or Chat-GPT. As I mentioned earlier, a useful framework I’ve been using for how to think about tools that improve developer productivity is **Abstractions** vs. **Aids**.

**Abstractions** – Abstractions are tools that make it such that developers do not have to think about a certain process or behavior in their program. An example for this could be a library  – say, for parsing dates. A developer has to know how to use the API, but the implementation details are abstracted away – it’s not important for the developer to know about these. Another example is a service like Stripe – where again, it’s important for developers to know what API to call and how their program is doing it, but the implementation for actual payments processing is done by a third party company.

**Aids** – Aids, on the other hand, make it easier and faster for developers to write code. Examples of this include both line-break debuggers and Github Copilot, but also lots of other tools developers use in their daily lives – text editors, code search tools, etc.

This is an important distinction because it’s important that we use our tools properly. At the end of the day, in order to actually make improvements to a codebase, fix bugs in a systematic way, and add new functionality efficiently, it’s important that developers read and understand what is in their codebases.

The power of **abstractions** is that they _reduce_ the amount of logic in your codebase, and reduce the surface area that developers working in a codebase have to understand. This is great, because it actually means that developers have to know less in order to make contributions in a codebase.

Aids can make it easier to build an understanding of code through making it easier to read and navigate your codebase. And of course, many aids help developers write code more efficiently through catching errors earlier, and tools like autocomplete that reduce the amount of typing required. However, aids, unlike abstractions, do not reduce the surface area the development team needs to understand. No matter how fancy your autocomplete is, as a developer, if you want to be able to make substantial changes to your code, you **have to understand what is going on**.

Thinking that an aid is an abstraction can fool you into thinking that you can understand less about your code, for instance, if you can simply line-break debug your way into finding a line of code that might be responsible for a bug, maybe you don’t need to read the surrounding context. 

But at the end of the day, even if this gets you to the right answer, your understanding of your code has not improved, and you are not in a better position to actually improve the code in question going forward.

----

I’m very excited about the future of programming – as time goes on, we are increasingly going to have better and better tools that both abstract away more and more, allowing us to focus on the stuff that matters for users of our programs. And we are also going to have tools that make the day-to-day of writing code so much better.

At the end of the day though, it’s crucial that we understand that if code is going in our codebases, that we do the work to understand it, and don’t use useful coding aids to skip that step.

I hope this distinction helps you understand your own tools better, and to think more about how to use developer tools appropriately.

Thanks for reading!