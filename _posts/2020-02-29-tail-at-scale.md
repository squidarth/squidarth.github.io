---
layout: post
title:  "Notes: The Tail At Scale"
date:   2020-02-29 14:04:38 -0600
authors: Sid Shanker
categories: article systems
---
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

I recently read the paper the ["Tail at Scale"](https://cseweb.ucsd.edu/~gmporter/classes/fa17/cse124/post/schedule/p74-dean.pdf)
from Google's Jeff Dean. It's super approachable, and a great introduction to how to
think about latency is large, distributed web systems.

## The Problem

In a distributed web system, a request might pass through many systems before returning with a response. In a basic model,
there might be a web application server and a database. Requests
are made to the web application server to get some data, which then makes a request to the database. The web application
server will then process the response from the database and return that response to the client.
If, during the duration of a particular request, either the web application server or database is down or unavailable,
the request *will* fail.

Many distributed systems will be far more complex than this, and a request must pass through 10s or possibly 100s of
different services before returning. As the number of services increases, the probability of one of them being unavailable
during the duration of a particular request goes up. If each service has a 1% chance of being unavailable, with the first model
I described, the chances of a request encountering either an unavailable database or unavailable web server is pretty low, but
with 100 different services, it becomes very likely.

There are many solutions to this problem in distributed systems--systems that are resilient to single services being unavailable
are called "fault tolerant". The "Tail at Scale" discusses a related, but different problem, which is that services, in addition
to having a probability of being unavailable for whatever reason, _also_ have a probability of being much slower than typical
for some small percentage of requests. Again, while a server being slow for .1%, or even 1% of requests does not pose a big
problem for systems with a small number of components, with a larger number of systems in the path of a single request,
the chances of hitting a slow path on some system becomes very high.

### Variances in Latency

Before diving into what solutions are possible for managing variances in latency, Dean spends some time discussing
the right ways to think about latency, as well as the reasons why not all requests to a system take the same
amount of time.

1. The right way to understand the latency of a system is by percentile, *not by* averaging requests. Think in terms of the amount of time
that  the p50 (the median), p90 (90th percentile), etc. requests take. This is because since one slow request might take 10-100 times
longer than the median, an average gives a fairly meaningless number. These higher percentile latencies are often referred to as "tail latencies".
2. Web systems are often run in multi-tenant environments where processes are often sharing resources on the same physical computer. This
means there is a lot going on on these servers that could result in momentary slowdowns in requests. For instance, garbage collection
might kick in during one particular request slowing it down, there might be contention for some shared global resource (like a network
card), or there could be some recurring job on the computer might have started. Other problems, like a server over-heating,
might also contribute.

So as an example, there might be a .1% chance of there being some event, either or hardware or software related that
causes requests to slow down. Given the number of requests that this affects, this probably wouldn't affect the p50 latency on that
machine, but would have an effect on "tail latency", the p999 (99.9th percentile). In reality, the numbers aren't so clean, but
this is why the p99 or p999 latency would be much higher than the p90, which would in turn be much higher than the p50 latency.

As discussed in the first section, as you increase the number of systems involved in handling a single request, the chances of
hitting the p99 latency on any of those machine goes up. And since a request can only be handled as fast as the slowest server
that handles on it, this will drag down the total performance of that request. With 10 machines, the chances of hitting the
p99 latency on one of those machines is:

$$ 1 - (.99)^{10} = 0.095 = 9.5\%$$

With 100 machines, the chances of hitting the p99 latency on one
machine is:


$$ 1 - (.99)^{100} = 0.634 = 63.4\%$$


This means that with 100 machines involved in the request,
requests will hit the p99 latency more often than not. This is pretty unacceptable performance for a web system.

## Some Solutions

Before diving into solutions here, it's worth thinking about what specifically we are trying to solve for. To restate
the problem, as the number of machines that a request has to touch before returning with a response goes up, the probability
of hitting the p99 latency of any of those machines goes up, and that will slow down the entire request.

We know, however, that each machine is capable of performing at its p50 latency. So the question becomes, in a distributed
system, how do we increase the chance of hitting the p50 latency of each machine in the path of a request?

Dean proposes a number of solutions, and I detail a couple of the notable ones here.

### Hedged Requests

The first option that Dean presents is having "Hedged Requests". The idea is fairly simple: let's say a request first hits machine A,
then B, then C. If A's request to B takes longer than some configured amount of time, simply fire off another request to B!

The thinking here is that  if A's request to B is taking longer than some threshold, then we have likely hit the p99 or worse latency
for that service. Between two requests, it's unlikely that they both hit a path with poor latency.

One of the downsides with this that Dean notes is that this *will* result in some amount of "wasted" work. After all, the results from
either the original request or the new request will be thrown away. Some of this can be mitigated through sending a signal once a response
comes back from either service to the other service indicating that it can cancel the request.

### Micro-partitions

Hedged requests help smooth over "tail" events, like garbage collection kicking in, or some other factor happening on
a service causing latency to increase temporarily . Another cause of variation in
latency is to do with the distribution of data in a particular system. In distributed systems where the quantity
of data is too large to sit on a single instance, data is often partitioned across a bunch of different machines.
For instance, in a simple scheme each partition stores some range of IDs. Typically, a partition will be replicated,
so there is set of replica machines in each partition.

Typical schemes partition data such that each partition has a roughly even amount of data, and such that each
partition fits on a single machine. In this approach, if data in a particular partition becomes "popular" for
some reason, and starts receiving more traffic than other partitions, traffic to that particular partition might
see higher latencies than other partitions.

To address this, Dean proposes "micro-partitions", in which partitions are sized so that they are much smaller than
what could be supported by a particular machine. These micro-partitions can then dynamically be moved around based
on traffic patterns. Therefore, if a particular micro-partition started getting a lot of traffic, it could be moved
to an instance with fewer other micro-partitions on it.

### Latency-induced Probation

Returning again to the problems addressed by hedged requests, some hardware or software related issues might
affect a particular machine for a period of time. For instance, if a single machine happened to have some
recurring job run it that consumed a lot of resources, it might process requests more slowly for potentially
a few minutes.

To address this, Dean proposes the idea of "latency-induced probabation" in which machines are taken out of
a cluster and put in "probation" if, for multiple requests, they perform poorly. Once a machine is on "probation",
it will no longer receive live requests. However, they will still receive shadow requests, which is real production
traffic that is duplicated and sent to these instance, but for which the response from the instance on probabation
will be ignored. If the latency on the instance in probabtion improves, it will be added back to the cluster.

It is somewhat counter-intuitive on the surface that reducing the size of a cluster temporarily might actually
*improve* latency!

#### What are "shadow requests"?

In the previous section I talk about the concept of "shadow requests". The general idea here is that on a normal,
live, production request, the request will go to a machine in the cluster, and return with the response. If an
instance is configured to receive traffic, that request, in addition to going to the machine that will give it
its response, will *also* be sent to the machine receiving shadow requests. This is typically managed by some proxy
service that manages sending the request to the "normal" instance and the instance receiving shadow requests, which
will know what response to return.

### A quick word on read vs. write requests

Something you might have noticed about some of the solutions noted in this paper is that the
some of them don't work if a request actually needs to make a change server side. For instance,
in the case of hedged requests, if a service just fires off a duplicate request, a particular
change might get made twice!

Similarly, in the case of latency-induced probation, if "shadow requests" are made for "write"
requests, they cannot be safely be made. These solutions don't work for write requests.

However, Dean points out that that in most web systems, write requests make a relatively smaller
fraction of all requests than read requests, and tend to be more tolerant to higher latency anyway.

## Conclusion

Thes are only some of the solutions Dean proposes--he lists out a few more in the paper. If you're
curious to learn more, I'd definitely recommend reading the paper--it's very accessible!
