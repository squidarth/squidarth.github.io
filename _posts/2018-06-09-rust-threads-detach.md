---
layout: post
title:  "Where do Rust threads come from?"
date:   2018-06-09 09:00:38 -0400
categories: rc rust concurrency
---

Last week, I wrote a [post](http://www.squidarth.com/rc/rust/2018/06/04/rust-concurrency.html) in which I discussed some of the things that I learned about Rust concurrency.

One of the things that I pointed out was that when you spawn a
thread within another thread, they _both_ have the main process
as their parent, which I demonstrated in this gif:

![rust_concurrency](/assets/rust-concurrency.gif)

After posting this on Reddit, I got the following question:

![reddit comment](/assets/rust-thread-comment.png)

This was a thought that occurred to me while I was writing the
post originally, but I didn't think much of it. This was just
"how it worked"<sup>TM</sup>. But, because I'm at the [Recurse Center](https://www.recurse.com/) and have the time to chase rabbits
like this, and out of desire to answer this person's question,
I decided to actually figure this out.

Before diving in, I do want to point out that I'm using Linux, and
the behavior of Rust _might_ different to what I'm describing here
on other platforms.

## Let's Revisit that Code Sample

Let's begin by revisiting the code sample in question:

```rust
use std::{thread, time};

fn main() {
  let duration = time::Duration::from_millis(3000);

  println!("\nMain thread");

  let handle  = thread::spawn(move || {
    println!("\nInner thread 1");

    let handle2 = thread::spawn( move || {
      println!("\nInner thread 2");
      thread::sleep(duration);
    });

    handle2.join().unwrap();
    thread::sleep(duration);
  });

  handle.join().unwrap();

  thread::sleep(duration);
}
```

While this seems like, it's fairly simple. We begin by spawning a thread, and waiting on that thread. That thread then spawns another thread.

While the intuition of the aforementioned redditor is that the parent
of "Inner Thread 2" ought to be "Inner Thread 1", when we examine
what's happening on the system using `htop`, we notice that that
isn't the case.

This has interesting implications, primarily, that it is possible,
although not the case in this example, for "Inner Thread 2" to
_outlive_ "Inner Thread 1".

## The Rust Docs

Let's start by seeing what the docs have to say about this:
From the [docs on threads](https://doc.rust-lang.org/std/thread/),
we see this:

> In this example [where a thread is spawned], the spawned thread is "detached" from the current thread. This means that it can outlive its parent (the thread that spawned it), unless this parent is the main thread.

Hm..ok so our understanding of the behavior is correct.

For the remainder of the post, I'll be focused on how this
behavior is achieved.

## Where to start?

In order to really understand what is happening here, we need to
get a better handle on what is happening with our system when
the `thread::spawn()` function call is run. Trying to read through
this code will probably require a lot of time and sorting through
the Rust standard library. Is there an easier way of figuring this out?

Well, let's start with what we know. We know that `thread::spawn()`
is creating a new OS thread that is managed by Linux. There is
probably some system call that's getting called to create that
new process.

## Enter: strace

If that's the case, then looking at the syscalls made by our
program will probably be pretty helpful. Thankfully, I've read
some of [Julia Evans' blog](https://jvns.ca/blog/2015/04/14/strace-zine/), and know that `strace` is a thing that exists!

For the uninitiated, `strace` is a program on unix systems that
can be run on a different program, and prints out all the system
calls that are made as the program runs.

Let's try running `strace` on our program to try to figure out
what is causing the new threads to get created. We start with

```bash
$ strace ./target/debug/concurrency-bp 
```

A lot of stuff gets printed out, mostly related to opening and
closing files:

```
sigaltstack(NULL, {ss_sp=0, ss_flags=SS_DISABLE, ss_size=0}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fd72978f000
sigaltstack({ss_sp=0x7fd72978f000, ss_flags=0, ss_size=8192}, NULL) = 0
write(1, "\nMain thread\n", 13
Main thread
)         = 13
futex(0x7fd7295700b0, FUTEX_WAKE_PRIVATE, 2147483647) = 0
mmap(NULL, 2101248, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x7fd7281ff000
mprotect(0x7fd7281ff000, 4096, PROT_NONE) = 0
clone(child_stack=0x7fd7283feef0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fd7283ff9d0, tls=0x7fd7283ff700, child_tidptr=0x7fd7283ff9d0) = 2318
```

But, admidst all of those calls are two that pop up around when
we print out "Inner Thread `" that seem relevant: `clone` and
`futex`.

However, there is only one call to each of those, despite the
fact that we are creating two threads.

After reading the docs on strace, it looks like we need
to use the `-f` flag in order to trace through the calls
of all the spawned threads from a process.

While we're at it, let's filter this list to just the
calls to `futex` and `clone`.

```bash
$ strace -f  -e clone,futex ./target/debug/concurrency-bp
```

## Clone

Let's start by taking a look at the `clone` syscall invocation:

```
clone(Process 2328 attached
child_stack=0x7fd565ffeef0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fd565fff9d0, tls=0x7fd565fff700, child_tidptr=0x7fd565fff9d0) = 2328
```

Reading through the [docs on `clone`](http://man7.org/linux/man-pages/man2/clone.2.html),
`clone` is a syscall that creates a new process is allowed to share the same "memory space"
with the calling process. This is in contrast to another Linux syscall used to
create processes, `fork`, which creates processes with their own memory space.
This is what the `CLONE_VM` flag above enables.

This property of "cloned" processes makes it a great candidate for thread
implementations.

The `CLONE_THREAD` flag that is passed to `clone` is really the one that
we're interested in here. From the docs;

> A new thread created with CLONE_THREAD has the same parent process as the caller of clone() (i.e., like CLONE_PARENT), so that calls to getppid(2) return the same value for all of the threads in a thread group.

Aha! So because Rust uses the `CLONE_THREAD` flag, the parent of "Inner Thread 2" is
set to be the _parent of "Inner Thread 1"_. If this flag were not to be used, the
newly spawned thread would instead have its parent be "Inner Thread 1".

It's worth noting here too that `clone` is the syscall used by the
[pthreads](https://www.ibm.com/developerworks/library/l-posix1/index.html)
`C` library, which is commonly used for creating threads in `C`.

## Futex

The next relevant syscall is `futex`. The [docs](http://man7.org/linux/man-pages/man2/futex.2.html)
describe it as a method that allows for a process to "wait" until a "certain condition becomes
true".

This is the syscall that is used to implement `join`. `clone` provides
the `CLONE_CHILD_CLEARTID` flag that performs a "wakeup" on a `futex`
when the cloned process terminates.
That address from `clone` can then be used in a subsequent call to
`futex` that will block the calling process until the "wakeup" is
performed by the cloned thread. Thus, with this call to `futex`,
the process that calls `join` will block until the joined thread
terminates.

## Conclusion

The main takeaway from this exercise was how `strace` can
save a bunch of time in terms of understanding how a
program interacts with the underlying operating system,
and saves you from having to trace through code to
get the answers you're looking for.

Also, the descriptions of syscalls and their APIs are
pretty readable. Don't be afraid to dig in and read these!

While this barely scratched the surface for how threads
work, hopefully you've been inspired to learn more,
and better understand how exactly your programs interact
with the operating system.
