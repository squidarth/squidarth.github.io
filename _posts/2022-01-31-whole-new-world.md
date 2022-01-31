---
layout: post
title:  "Rewatching 'A Whole New World'"
date:   2022-01-31 00:00:38 -0400
authors: Sid Shanker
categories: programming
---

I just rewatched Gary Bernhardt's incredible 2012 talk ['A Whole New World'](https://www.destroyallsoftware.com/talks/a-whole-new-world). This is absolutely one of my favorite tech talks
of all time -- and it holds up. Before I start
gushing about this talk, unlike your average tech talk, this talk _has a twist_.
So you might want to give it a watch before reading any further!

[Watch 'A Whole New World'](https://www.destroyallsoftware.com/talks/a-whole-new-world)

I watched this talk for the first time several years ago, when I was much
newer to the industry. Now that I've been around a little longer, and especially now
that I work in software infrastructure, this talk hits a little differently.

In the talk Gary tells us that he made a new terminal-based editor,
with some fancy functionality, like being able to have exceptions from production show
up inline in the editor, or having Graphviz-like diagrams of your code render.

Of course, much of this functionality would not be possible in a terminal-based editor.
So, Gary tells us that in order to do this, he made a new Terminal, with features
such as being able to draw raster graphics.

The twist, of course, is that he actually did not build either of these things. It was
a thought experiment, cleverly designed to elicit a particular response from the audience.

The overwhelming emotion that most engineers have seeing this talk is one of **surprise**. Now,
for starters, it's a little weird that that would be the prevailing emotion, given
how the tech industry is supposed to be all about innovation. But in the remainder of the
talk, Gary digs into why we feel surprised, and what that says about the industry.

## Incrementalism can only get you so far

One of the core reasons why a new terminal-based editor, or a new terminal would
be so _surprising_ is because of what Gary calls "Shipping Culture". He quotes Seth Godin
here, who captures a lot of the thinking in tech with his "Ship Often" mantra. The thinking
here is that through bringing products to market _faster_, we can learn quicker from
customers, iterate. And the need to ship quickly biases us towards incremental changes.
Building a new editor or terminal from scratch, on the other hand, is the opposite of incremental.

I'll add another reason for the bias towards incremental changes -- it is much easier
to form hypotheses around them and quantitatively understand their impact. This ultimately
makes it much easier to justify (to yourself and others) working on incremental things,
than working on a more substantial change, where making a quantitative claim about the
impact might be harder. 

I've certainly noticed this bias in myself, especially since I've spent much of my
career working at startups, where "Shipping Culture" is core to survival. This has sometimes
manifested in being dismissive of projects that involve substantial, fundamental changes that
will take a while, as opposed to ones that deliver value quickly.

The problem with this kind of thinking is that with existing tools and infrastructure,
we ultimately hit limits where to make further progress, we will have to replace more
fundamental pieces of infrastructure. If we are to ever have a better terminal, that will
ultimately require engineers setting out to build such a thing, and will not be the
result of incremental changes.

## Not recognizing when things are bad

Other thing Gary points out that hits close to home is that when
we've been working for a while, it's often hard to see what
parts of our tools are actually bad or confusing. The problem here is that it _may not
even occur to you that there there's anything wrong with your editor/terminal_!.

A recent-ish example of this for me is `git`. I've been using it since I started
programming, and is almost an invisible part of my workflow at this point. There
are a lot of usability problems with it though, that hadn't even occurred to me
because I'm so used to the tool. A big example here being that `git checkout` does
a _lot_ of different things. Or how difficult and scary it is to undo a commit.
A lot of this didn't occur to me until I came across the [Gitless](https://gitless.com/) project,
which aims to simplify the naming of commands. 

This [blog post about the issues with SQL](https://www.scattered-thoughts.net/writing/against-sql/)
is another example of something that has almost become invisible because of how
much time I've spent with it.

I think my big takeaway here is to keep a beginner's mind when using your tools. This
is also something that becomes apparent when working with more junior folks -- take
note of the things that are annoying and confusing!

## A Modern Example

This talk is from 2012, and I think a lot of the things that Gary has to say about the
culture of the tech industry still ring true today. However, there have in the last 10
years been some attempts to replace fundamental pieces of infrastructure, that
follow the model that Gary talks about at the end of his talk.

The best example of this is [the Rust programming language](https://www.rust-lang.org/). 
Rust is a programming language that aims to bring memory safety to systems programming.
Because the language is memory-safe, it eliminates a whole class of bugs, making Rust
code much more secure than non-memory safe languages, like C.
And because it has very little overhead (ie: no garbage collector), it can be used in a lot
of contexts where C is commonly used without a major performance hit.

It's being adopted all over the place, and there it is now
[supported in the Linux kernel](https://www.zdnet.com/article/rust-takes-a-major-step-forward-as-linuxs-second-official-language/).

Switching programming languages is a major change, especially for projects
like Linux, which need to be conservative around the changes they make. However,
this is a clear case where incremental improvements were never going to eliminate
memory safety bugs entirely.

## Conclusion

I think this is a really important talk for developers to revisit every
once in a while. It's a great reminder to break us out of common thought patterns--
both biases towards incremental improvements, as well as blindness to inefficiencies
and problems with our tools.

Highly recommend giving it a watch if you haven't!