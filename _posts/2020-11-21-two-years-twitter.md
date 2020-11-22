---
layout: post
title:  "Two Years at Twitter"
date:   2020-11-22 12:00:38 -0400
authors: Sid Shanker
categories: career
---

A couple weeks ago, I hit two years at Twitter! Given that I'd spent most
of my career before working at fairly tight-knit startups with < 50 person
(and mostly co-located) engineering teams, the last couple years have been
quite a departure.

Overall, I've gotten to build a lot of cool software, and have learned a lot
about software engineering and how to operate in the context of a larger company.
I thought I'd share some of these lessons! These might be particularly interesting to
folks who are considering or about to go through a transition from startup-land to big company-land.
I choose to focus on non-technical skills I've learned.

## What I've Been Up To

I work on the User Service team at Twitter--we're an infrastructure team that
owns the services that serve real-time data related to _users_ and _relationships between users_.
As one might imagine -- being able to serve this data is crucial to the site staying up,
so reliability is a big component of what our team needs to work on. Our main mission now,
however, is figuring out how to help product teams build new features that involve user & relationships
more easily.

A lot of the lessons I've learned in the last couple years are specific to infrastructure teams, but
I'm sure of these translate.

## Focus on High-Leverage Work 

Given where my team sits in Twitter's stack, we have a *lot* of requests from
other teams for changes the services that my team owns. These changes might be related
to features missing to build some new product feature, or from some team that owns a library
that we us asking us to migrate to a newer version. Deciding where to focus our time, on an infrastructure
team is challenging!

Early in my Twitter career, my instinct was to be as helpful as possible. When a product team
wanted to build something, and tacking something onto the services that my team owned was required,
my instinct was to build that missing piece, and continue advocating for us to do this kind of work.
After all, it *sucks* to be a blocker. Furthermore, I was used to operating in the **context of a startup**,
where the maxim was always to do the smallest piece of work required to ship the product you are trying
to ship.

However, some feedback that I received on this was that continuing to do this work was actually
**not** the highest leverage work I could be doing. While it felt good to be helping support these product
teams, by continuing to do this work, we were ensuring that our team would continue have to be kept in the
loop for future product development. This is long-term not good for the company, as it means that the number
of teams required to build something stays high.

The _alternative_ to doing this kind of work, would be, for instance, to re-architect how our services to
work such that product teams could make the changes they require _themselves_, rather than having to
ask us to make these changes. A project like this is more high leverage, since it reduces the amount of time
and people required to ship anything, and also because it _frees up our team_ to do other kinds of work
in the medium and long term, rather than having to continue to support product teams.

There are few lessons to draw here:
1. Be clear about what the objective of your team is

In this example, I was very interested in helping Twitter ship specific product
features very quickly, rather than being focused on making the changes these
product teams needed to make be "self-serve". It's important to be clear about
what exactly the objective your team is driving towards, and let that be a framework
for what work you end up doing. While my instincts of "let's build the smallest thing
we need to get this to work" were fair instincts, it was my definition of "this" that was off.
Our job as a team is not to directly help ship features, but rather build things that make
life easier for those building those features.

2. It is possible to be too nice

There is a cost to being too accommodating to customers' requests. Again, if all
we did was support product teams building their features, we would never have time
to work on projects that help us get out of doing this work, which would ultimately
make life for these product teams easier.

## Executing on Large Projects

Something else new to me at Twitter was working on large projects, involving lots
of stakeholders. I've learned a lot about how to do this effectively--here are some of the
highlights: 

### Finding the Right Scope for a Project

When building software, there are a lot of changes one *could* make. For instance, let's say you are
building a new project feature, and building that feature requires touching a piece
of code that is scary to change. Should that piece of code be refactored as a part
of this project?

Because of this nature of software, scoping software projects appropriately
is challenging!  The biggest lesson I've learned about this is to be extremely
clear about what problem is being solved with a new project. Design documents
and RFCs (which I go into in more detail later) are a really good way of
articulating the goals of a project.

This is important, because it grounds any further discussion of what ought
to be done in a project. Once you have articulated what the objectives are,
and what _business value_ is captured by the project, you can enumerate the planned
work, and determine the fastest route to capturing the value that's been described.
In the original example I gave, it could still be possible that doing the said
refactor _would_ be the fastest path to capturing the value of the new feature
that is being built. It also might push out the time it takes to capture that value,
and add risk of its own, and if that's the case, it might be worth descoping
that refactor from the project.

The problem in not having a clearly articulated objective is that it becomes easy for
extra work to become part of a project, causing it to get dragged out, and lead
to _poor morale_ on the project, as well as a belief that "things will never get done".

A concrete and illuminating example was at a previous job (not Twitter) of mine. We were working
on our deployments, and ended up spending a bunch of time trying to figure out how to do "zero-downtime"
migrations, since somebody at the company had mentioned that the site "should not go down".
Here, the expansion of scope to include "zero-downtime deploys" was due to a misunderstanding
of requirements & business objective of the project. Guaranteeing 100% availability is not
realistic, and we probably should have discussed whether the amount of downtime our migrations
were causing were unacceptable or not, before embarking on that.

### Writing Design Docs

Twitter is the first company I've worked at with a very strong design culture. There's
an expectation that any non-trivial projects that are undertaken have a design doc written
that explains both the **why** behind the project, as well as the **how**. I've found that
writing design documents or RFCs is a very good exercise for figuring out what needs to be built. 
I've noticed that in the process of writing these, I end up with a much better understanding
of the systems that I'm working with, as well as the problem I'm trying to solve. This deeper
understanding then helps inform what parts of a project might be the most difficult or where
risks are.

One of the things that Twitter does well here is having templates for different types of design
docs. Going through the template forces you to think about different angles of your project.
Have you thought about capacity planning for this? What work might be required of other teams?
What's your strategy for rolling out this change?

I've found writing these docs has done a lot to clarify my thinking about projects that I embark on,
and I think that the smaller companies I've worked at would benefit from having more of a design culture.

## Adjusting to Remote-First

Like for so many other teams, adjusting to Pandemic-era life was definitely challenging. However, adjusting
to remote wasn't challenging exactly in the way that I expected. For instance, I worried that there
would just be a time wasted from not being able to ask people questions and have adhoc discussions live. This ended up not being
as big a problem as I expected -- I've found that having to write out questions is a good exercise that itself
ends up saving time.

The real challenge has been helping people feel connected. The main lesson here -- it's hard to motivate people to come
to ad-hoc Zoom happy hours, and especially hard to keep these going. On the other hand, experiments with book clubs
and games of [Among Us](http://www.innersloth.com/gameAmongUs.php) have been very successful!

## Some things I've read

I've also read a lot of books and blog posts over the last couple years related to adapting to
the context of a larger company. Here are some of the highlights:

* **[Google SRE Book](https://landing.google.com/sre/books/)** -- this book is a great resource for understanding how to operate
services at scale. Practically, the book will help you understand a lot of the vocabulary that is used at bigger companies.
* **[Manager's Path](https://www.oreilly.com/library/view/the-managers-path/9781491973882/)** -- Haven't alluded to this in this
post, but if you're curious about management careers, this book is a great resource
* **[Righting Software](https://www.oreilly.com/library/view/righting-software-a/9780136524007/)** -- A coworker recommended this to me--
while the "managing a project" section I didn't find as helpful, the first part of the book on software architecture is fascinating
* **[Will Larson's (@lethain) Blog](https://lethain.com/)** -- lots of great stuff here, especially love the latest posts on [engineering
strategy](https://lethain.com/engineering-strategy/).

## Conclusion

So far, I've really enjoyed working in this new environment, and am proud of what I've been able
to accomplish so far. I'm also very grateful for my team for being awesome people to work with,
and for all of the helpful feedback <3.

Please feel free to reach out (I'm [@sidpshanker](https://twitter.com/sidpshanker) on Twitter) if
you have any thoughts or have had similar experiences!