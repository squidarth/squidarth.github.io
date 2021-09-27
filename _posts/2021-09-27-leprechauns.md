---
layout: post
title:  "Notes: The Leprechauns of Software Engineering"
date:   2021-09-27 00:04:38 -0400
authors: Sid Shanker
categories: software-engineering 
---

I've recently been frustrated by the lack of
rigor in discussions of software engineering practices.
For any topic that might trend on programming Twitter (How should we interview people?
Are integration tests worth it?) you'll find hundreds of
opinions, often informed by little more than personal opinion.

A feeling that a lot of these questions ought to be informed
by **empirical research** led me to spend more time reading about
academic Software Engineering Research. I inevitably wound up on [Hillel Wayne's](https://twitter.com/hillelogram) [blog](https://hillelwayne.com/), and ended up
discovering the book [Leprechauns of Software Engineering](https://leanpub.com/leprechauns) by Laurent Bossavit, one of the most interesting pieces of content on this topic.

The book examines academic Software Engineering Research and both explains
how research is done today, and asks whether **in the current research paradigm, we can really trust the results being produced**.

In the process, Bossavit both gives readers a lot of really interesting history about how certain
beliefs in the field became widespread, and also teaches readers a framework of skepticism towards research.

For anyone interested in how to move discipline of Software Engineering forward, I'd definitely
recommend picking this book up!

_Note here that "Software Engineering Research" is a subfield of Computer Science that studies
the practice of building software, and not Computer Science as a whole._

## Where do facts come from?

One of the core missions of this book is to examine "common knowledge" in the
industry and understand the _origins_ of that knowledge. Some of the beliefs he goes through are:

* The 10x engineer -- the idea that some people are orders of magnitude more productive
  in development than others
* Test-driven development (TDD) helps reduce bugs & development time
* Bugs found earlier in a project lifecycle are easier to fix

For each of these beliefs, he digs into the empirical research that industry folks
cite when they invoke them, and questions how much credence should place in them.

In the case of the 10x engineer belief -- he digs into the papers that are often
used to cite this claim. He finds the original paper that mentions this claim,
written in 1968, involving students working on a particular debugging task. 
Despite the criticism of the paper at the time, including the fact that not
all of the students were working in the same programming environment, this
paper has been cited again and again as evidence for the existence of this 10x gap.

Many of the other papers he finds have a number of problems. A common problem is that
a lot of studies around programmer productivity involve studying _different tasks_, making
apples-to-apples comparisons difficult between studies. Other papers do not cite any
empirical evidence at all, or just **cite earlier papers**, interpreting weaker
claims made by earlier papers as facts.

All this to say, even when taken in aggregate, the research available on this
topic on it's face doesn't really support the 10x claim, on the terms that people in the
industry operate in. 

## Rejecting the Premise

Bossavit goes even further in his criticism of research on programmer productivity,
which is to say that none of the papers he looked at did any introspection about the
question they were asking! What exactly is meant by programmer productivity?

This is a really complicated question. First off, "programming" is not a singular
task -- working on an operating system is very different job to working on a web
app. Secondly, even if we were to focus on a single problem domain in software
engineering, business contexts are really varied. The needs of a startup attempting
to get to market really quickly are different to the needs of a team working on designing
software for a spacecraft that can't be updated for decades after launch. How can we
define productivity in a general way that works in both of these environments?

Additionally, even in the absence of a rigorous, generalizable definition of
productivity, environmental factors are huge factor in our perceived productivity.
How much sleep did programmers in a study have? What were the work environments like?

The main takeaway here is that a major problem with empirical software engineering
research today, which often feeds into commonly held beliefs, is there no shared
understanding of the phenomona being studied.

## Software Engineering Research Paradigms 

This pattern where researchers don't introspect about the subject they are studying
holds for other questions too. One of the other main claims Bossavit spends a lot
of time on is the claim that "Defects found earlier in a project lifecycle are
cheaper to fix than those found later".

My reaction to this claim, based on my own experience, seems plausibly true! However,
the same problems with the 10x claim hold here. Do we have, across the industry, a
common understanding of what a "defect" even is? And how do you go about quantifying
what the cost to fix a defect is?

Bossavit argues that the current paradigm of research is really based on how research
into other professional disciplines, like medicine, is done. In medicine a lot of research
into the merits of practices aims to answer questions like "does handwashing reduce the number of infections?". Software engineering research takes a similar approach, aiming to evaluate the merits of practices (say TDD), against outcomes.

The difference, however, is that in software engineering, the very things that we are studying
are human creations, not properties of the universe! Bossavit argues that a lot of research
has the implicit assumption that "code", "bugs" and related concepts are _properties of the universe_ to be studied, rather than the human-created constructs that they are.

## A Way Forward? 

Early in the book, Bossavit points out
that very few working software engineers pay attention to or read journal articles about
software engineering. While there are obviously other factors that go into that,
there's some indication there that our current paradigm of research is _not yielding results
interesting to practitioners_.

So what would need to change then to produce more useful results? Bossavit admits that it's unclear
what the next steps need to be in order to actually start making progress. However, he does a
good job of outlining what needs to change -- specifically, to stop treating the way we
work with computers today today as immutable properties of the universe, and to start
acknowledging the *human factors* that go into our work.

I don't have enough context on the whole field of Software Engineering Reseach to judge
whether this is totally fair or not -- I imagine that a lot of this work is probably happening!
But I am convinced that more introspection about what studies are aiming to actually study
is important! 

## Conclusion

While the book in some ways left me with more questions than answers -- I definitely came away from reading
_Leprechauns_ with a much firmer grasp of why Software Engineering is such a hard field to study.
And I definitely feel like I got answer to my initial question of whether the big debates
in Software Engineering could be settled with _data_ -- it feels like today, we lack the methods to really do this.

If you're a working engineer or interested in the field at all -- I'd highly recommend
giving _Leprechauns_ a read!
