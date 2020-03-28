---
layout: post
title:  "Notes: Understanding Real-World Concurrency Bugs in Go"
date:   2020-03-27 14:04:38 -0600
authors: Sid Shanker
categories: article systems pl
---

The next paper I'll be writing about is ["Understanding Real-World Concurrency Bugs in Go"](https://songlh.github.io/paper/go-study.pdf),
from Tengfei Tu, Xiaoyu Liu, Linhai Song, and Yiying Zhang of Penn State and Purdue University.

This paper explores a fundamental claim of the Go programming language, that the language features around concurrency
make it harder to introduce bugs, and evaluates that claim against empirical results. The authors of the paper studied actual
bugs found in open source projects and their fixes, and find that in fact, these new language features don't
*necessarily* make it harder to introduce concurrency bugs.

This paper was particuarly exciting to me for a few reasons--first off, a lot of discussion of design
patterns in computer science don't actually include any empirical results (ie. using this pattern
_actually_ prevented bugs in these situations), and this paper demonstrates how a study like this
could be done. Furthermore, in doing so, the paper gets into the human factors of programming language design
that end up leading to there being bugs. I also think a lot of the language of the paper could be useful to
software engineers. Finally, I learned a lot about concurrency through reading about the bugs in the paper.

In this blog post I'll cover some background on Go and concurrency, and then
dive into the findings of the paper.

# Some Background on Go and Concurrency

The paper begins with some background on both concurrency and how Go handles it.

## What's a goroutine?

The core concurrency construct in the Go programming language is the "goroutine". A goroutine
is a function that can run concurrently in Go. Unlike threads, these are managed by the Go runtime
itself--a new thread is *not* spun up for each goroutine. Therefore, you can theoretically have millions
of goroutines running on a single machine, and don't have the same limits you would have would traditional
threads.

Goroutines are easy to create, and can be created by adding the `go` keyword before
a function or anonymous function:

```go

func foo(msg string) {
  fmt.Println(msg)
}

func main() {
  go foo("bar") // starts a go routine
  go foo("baz") // starts another go routine
}

```

## Coordinating Goroutines

Similarly to threads, goroutines have different mechanisms for coordinating behavior.
The authors of the behavior distinguish between what they call "Shared Memory" access
and "Message passing".

### Shared Memory Access

Shared memory accesses are what a lot of traditional programming
languages use to coordinate behavior between threads and involve structures like Mutexes,
which coordinate access to shared resources, and Condition Variables, which allow threads
to wait until certain conditions are met to continue processing.

Here's an example:


```go

func foo () {
  p.mux.Lock()
  // Note that the `defer` keyword in go executes the following code
  // once the surrounding function completes.
  defer p.mux.Unlock()
  // Do some stuff
}

```

In this code, all the behavior in `foo` after the `p.mux.Lock()` is wrapped by a mutex, and therefore
can only be executed by a single goroutine at a time.

### Message Passing

Message passing is a coordination mechanism for goroutines that involves
a language-specific concept called "Channels". The general idea is that
channels are a communication construct that goroutines can write to and read
from. The key to these being effective is that both sending information to a channel
and receiving data from a channel are blocking operations. Here's an example:

```go

func main () {
  c := make(chan int)

  go func() {
    sum := 0

    for i := 0; i < 10; i++ {
      sum += i
    }
    c <- sum // write `sum` to `c`
  }


  final_sum := <- c // read value from `c`
  fmt.Println(final_sum)
}
```

In this example, the parent goroutine is coordinating with
the child goroutine using the channel `c`. It's worth noting
that the child will block until the line `c <- sum` until
the parent reads the value from the channel.

### Are Message Passsing and Shared Memory Interchangeable?

Something that isn't explicitly spelled out in the paper but
I think is useful background to have is that it's possible
to accomplish the same tasks with shared memory techniques like
Mutexes and Condition Variables with channels.

See [this post](https://medium.com/dm03514-tech-blog/golang-monitors-and-mutexes-a-light-survey-84f04f9b7c09)
for some examples of how to implement a "monitor goroutine" to protected
shared memory using channels. The post also discusses some of the
pros and cons of both approaches.

# The Paper's Methodology

With some background out of the way, we can finally move onto the study itself! To reiterate,
the goal of the study is better understand whether the new concurrency primitives that Go
introduces actually help prevent bugs. The paper also discusses solutions to each of these
bugs, and leaves open the possibility by the end of the paper to build more automated tooling
to prevent & automatically fix some of these bugs.

## Sourcing Bugs

The first challenge in any study like this is to have real bugs to study. Thankfully,
there are some very large open source Go projects, each of which with an issue
tracker. The authors of the paper used Docker, Kubernets, etcd, CockroachDB, gRPC, and BoltDB,
and searched for bugs in those projects that seemed related to concurrency. They took the bugs
that were related to concurrency, attempted to reproduce them, and studied the fixes
for those bugs.


## A Bug Taxonomy

With bugs in hand, the next big challenge was coming up with some means of classifying
bugs & identifying causes. The authors divide bugs across two axes. The first is the *behavior
of the bug*. The two possible "buggy" behaviors that the authors classify the bugs into is
whether the bug was "blocking". A "blocking" bug is one in which a goroutine is unable to progress
because it is blocked on a lock or a channel. A classic example of a blocking bug is a deadlock--where
two goroutines are each waiting for the other to release some resource to continue. The other type
of bug is a "non-blocking" bug. In these bugs, all goroutines are able to proceed, but produce
some other kind of non-desirable behavior (they just do the wrong thing).

The reason it's meaningful to make this split is because the solutions are very different--
we'll get into this later, but it's possible to have static analyzers detect deadlocks, and
other blocking scenarios.

The second axis the authors use to divide the bugs is the  *cause of the bug*. This I've
already alluded to already, but the two causes are 1. Use of Shared Memory, and 2. Message Passing.

For each of the quadrants, the authors do a deeper dive into the specific causes of bugs and their solutions.

## Solutions & "Lift"

Another aspect of the study is studying the solutions to the bugs as well, to better
understand how common bugs could be more easily detected and potentially automatically solved.
For each of the solutions that are identified, the author measures the *lift* of a
particular solution. *Lift* is a measure of given the root cause of a bug, what is the
probability that the proposed solution will fix it? For a concrete example, bugs related
to a Mutex are often fixed by a "Move" operation (which refers to moving a lock operation),
resulting in a high *lift* value for that bug/solution combination.

This is an interesting property of bug fixes to study, because, as the authors note,
high *lift* values suggest the possibility of automated fixes to these bugs.

# Findings

As I mention above, for each of the quadrants in the authors' bug taxonomy, the authors
do a deep dive into specific bugs that they found and how they were resolved.

While I don't go into those details here, if you are interested in understanding
concurrency better, I'd highly recommend reading the relevant sections of the paper!

## Blocking Bugs

For blocking bugs, what the authors find is that message passing, and using the new
channels construct in Go and actually contribute to more blocking bugs than using
shared memory. Most of the bugs caused by message passing are due to misunderstanding
how message passing works.

The authors also find that blocking bugs caused by message passing are harder to detect.

On the other hand, the blocking bugs caused by shared memory typically have the same
causes and resolutions as they do in traditional languages, and that these fixes are
better understood than fixes to bugs related to channels.

### Solutions to Blocking Bugs

The bright side for blocking bugs is that most blocking bugs have relatively simple
and easy-to-characterize solutions. In other words, solutions for bugs in this category have a
high *lift* value.

## Non-Blocking Bugs

The findings in the cases of the non-blocking bugs are very different,
as the authors find that there are more bugs caused by traditional shared memory
than bugs caused by message passing. Furthermore, many of the bugs caused by
traditional shared memory patterns are actually fixed by switching to using
message passing.

The caveat here is that many of the message passing-related bugs were harder
to discover & fix than the traditional memory-related bugs. Also, many of the
bugs related to traditional memory were the result of libraries that are easy
to misuse.

### Solutions to Non-Blocking Bugs

In the section on fixes for the non-blocking bugs, the authors note that many
of the bugs related to traditional memory were caused by misuse of shared variables,
or inadvertently sharing data between goroutines. The most common solutions to these
issues were using Mutexes to protect access to shared data and switching to using channels to avoid
having to use shared data at all.

There were some solutions that had high *lift* values in this section too, some notable ones being
the cause "misusing channel", and solution fix "fix channel", and the cause "anonymous function",
and the fix strategy "make data private".

The authors also note in this section that automatic detectors, like data-race detectors, do not do an effective job of
detecting all of the types of non-blocking detectors.

## Potential Issues with the Study

The authors call out some threats to the validity of the study at the start, where they note that
they were only able to use bugs that they were able to reproduce in these open source projects. There
were not a lot of these bugs (the amount of data in the study is fairly small), and a lot of the bugs
were not reproducible and therefore excluded from the study.

An issue that wasn't mentioned but I think still stands is that the comparison between shared memory-related
bugs and message passing bugs is not necessarily a fair comparison. For instance, comparing a bug caused by
shared memory and a bug caused by memory passing doesn't take into account the *incidental complexity* of
the logic of the code. In other words, in this study, we don't have side-by-side comparisons of the same
logic written using shared memory and the *same* logic written using message passing. It's possible that
a bug caused by message passing was just a hard piece of logic to write and that the programmer, if using
shared memory would have written the same bug.

It's of course, hard to control for that, and I think the investigations of the various root causes
of these bugs helps remove some of the uncertainty around this.

# Takeaways

There are some really interesting takeaways from this paper. One big takeaway is that a lot of the new
features of Go that were intended to help reduce the number of concurrency bugs actually *don't accomplish
this*. The authors note that a lot of these issues are due to users of the language misusing these
language features. I think that what this suggests for language & library designers is that there
is always a risk when introducing new concepts that existing developers won't necessarily be familiar with.
It's clear from the non-blocking section that message passing *can* help prevent some types of bugs,
but in the blocking section it's clear that it doesn't just do this out of the box, it's important
for developers to know how to use it safely.

This takeaway opens up another interesting question--which is that design for "pros" and design for
"beginners' doesn't necessarily look the same. In this case, message passing can clearly make
it harder to produce concurrency bugs in some cases, but for those less familiar can actually
cause more.

Another interesting takeaway for me personally comes from the actual methodology of the paper.
I'm new to research in software engineering in general, and I found the exercise of studying
and classifying bugs very interesting. I wonder if it would be valuable for companies
to do analyses like this of bugs in their codebases to actually in a data-driven way figure
out approaches to reducing defects in their software.

Furthermore, I learned a lot about concurrency in the process of reading this paper. This
general approach of categorizing real bugs that actually happened and documenting them
is a fantastic way to learn about concepts in software engineering.

# My Open Questions

This paper left me with a couple open questions. One is a point that I raised above, which
is whether understanding the *incidental complexity* of a particular piece of code
has implications for comparing bugs with different causes.

The other main open question I had is whether or not the choice of using traditional shared memory
or message passing had any implications for automated unit or integration testing.

As an aside, for anyone interested in more detail about the bugs studied in the paper, they
are all available on Github [here](https://github.com/system-pclub/go-concurrency-bugs).

# Conclusion

I had an awesome time reading this paper. If you're interested in dipping your toes into software
engineering research, this seems like a great place to start. If you're interested in concurrency,
I think this paper illuminates a lot of the challenges in writing correct concurrent code.
And finally, I think that a lot of the language used around analyzing bug and their solutions
seems like it could be useful in general software engineering work.

Give it a read!
