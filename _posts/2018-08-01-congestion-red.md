---
layout: post
title:  "Congestion Control III: RED"
date:   2018-08-01 12:00:38 -0400
categories: rc programming networking
---
<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

In my [last](http://www.squidarth.com/rc/programming/networking/2018/07/18/intro-congestion.html) [couple](http://www.squidarth.com/rc/programming/networking/2018/08/01/congestion-cubic.html) posts on TCP congestion control, I discussed strategies that TCP senders
can use to handle congestion and figure out the optimal rate at which to send packets.

In this post, I'm going to change it up and talk about strategies that *routers* can use
to prevent congestion. Specifically, I'm going to focusing on an algorithm called Random Early
Detection (RED).

As usual, I put together a [Jupyter notebook](http://squidarth.com/Link-level-Congestion-Control.html) documenting my results and explanations.

# Recap: Congestion Control

As I've mentioned in previous posts, congestion happens in computer networks when senders
on a link send more packets to the link than it can handle. As senders on a particular
link send more packets to the link, if the link cannot accomodate the quantity of packets
being sent, the router connected to that link will begin queueuing up those packets. If the
queue on that router fills up, subsequent packets arriving at the router will get dropped.

When this situation occurs, TCP senders need to reduce the rate at which they send packets.
There are two indicators that TCP senders have that congestion is happening:

1. Packet drops
2. Increased round trip times

While senders could theoretically use both of these signals to control the rate at which they
send packets, in reality, many TCP implementations only use packet loss.

## Why might this be an issue?

If TCP senders only react to packet loss, that means that if they are sending too many packets,
they only find out once the queues at the routers fill up. In cases
where there is a sizable queue, that queue will delay the time it takes for the sender to
realize that congestion is happening. In the case of [CUBIC](http://www.squidarth.com/rc/programming/networking/2018/08/01/congestion-cubic.html), the default congestion
control algorithm on Linux, the sender will ramp
up its window will grow its window really fast, fill up the queue buffer, see a bunch of
packet loss, and then drop its window size very quickly.

![cubic_high_bdp]({{ "/assets/cubic_high_bdp.png" | absolute_url }})

In this world, it becomes very hard for CUBIC to fully achieve its potential, because it gets
signal about how much congestion is happening too late. It makes the link kinda like the
hot water knob in the shower--the feedback happens after you've moved the knob, and you don't
realize if you've made it too hot until later.

### More on the queues

It sounds like from this that the queues are masking congestion issues, and possibly even
making the problems these links experience worse. So why not just make queue smaller?

I think to understand this requires an understanding of why these queues exist in the first
place. The reason we want routers to have queues is because traffic isn't consistent--it's possible
that in one minute there's a sudden burst of traffic because a bunch people decided to visit a
website at exactly the same time. If links dropped packets immediately after the link capacity
is full, they would have no resilience to bursts of traffic. Queues can serve as smoothing
functions that can absorb bursts of traffic and hold them while the link is being emptied.

Throwing away the queues or making them smaller makes them less resilient to bursts of
traffic.

# Can you manage queues differently?

Alright, we've established that queues are important, but that they can mask congestion problems.

It turns out that changing the way the queue is managed can preserve the benefits of having
queues, and also solve the congestion masking problem. This general approach of changing
how a queue handles filling up is called [Active Queue Management](https://en.wikipedia.org/wiki/Active_queue_managementa) (AQM).

When I talk about how queues are "managed", I'm specifically referring to how they handle
being too full. The traditional, intuitive approach to handling this is called droptail.
The way droptail queues work is that they fill up until they hit capacity. Once they
hit capacity, all subsequent attempts to add packets to the queue fail.

However, queues don't *have* to wait until they are full to begin dropping packets.

## Introducing RED

Random Early Detection (RED) is a queueing discipline that drops packets from the queue
probabilistically, where that probability is proportional to how full the queue is. The more
full the queue is, the higher the probability of a drop is.

![red_gif]({{ "/assets/red_gif.gif" | absolute_url }})

Here we see that as the queue is emptier, packets don't get dropped, but as the queue gets more
full, it begins to drop packets with more frequency.

## Drop healthy packets?

It's a little counter-intuitive that dropping perfectly healthy packets arbitrarily would
lead to better performance. The intuition though, is that dropping packets earlier provides
an indication to TCP senders that they should reduce their congestion windows. This is a much
better situation than letting the senders fill the queue entirely, experience a whole series
of drops, and then have to dial down their congestion windows a lot.

# Results

In every scenario that I ran, RED performed better than droptail. While the differences were
more subtle on low BDP networks, there was a pretty dramatic improvement on high BDP networks.


![cubic_high_bdp_droptail]({{ "/assets/cubic_high_bdp_droptail.png" | absolute_url }})

*CUBIC running on a high BDP network with droptail*

![cubic_high_bdp_red]({{ "/assets/cubic_high_bdp_red.png" | absolute_url }})

*CUBIC running on a high BDP network with RED*

The intition for why RED performs so much better here is that in the droptail case, CUBIC
is constantly alternating between filling up the queue, and then dramatically dropping the congestion window when a bunch of drops happen, while in RED, drops happen earlier, resulting in
fewer consecutive window size reductions.

This is apparent in graphs of the link queue sizes in each of these scenarios:

![link_queue_size_droptail]({{ "/assets/link_queue_size_droptail.png" | absolute_url }})

*Link queue size using droptail*

![link_queue_size_red]({{ "/assets/link_queue_size_red.png" | absolute_url }})

*Link queue size using RED*

# Some Implementation Details

I worked with a fellow [Recurser](https://recurse.com), Venkatesh Srinivas on actually writing
an [implementation of RED](https://github.com/ravinet/mahimahi/pull/122) in a [network simulator](http://mahimahi.mit.edu/) I've been using for
congestion control experiments.

The [RED Paper](http://www.icir.org/floyd/papers/early.twocolumn.pdf) has a full section
on the recommended implementation. One of the most important takeaways from this is that
when computing the probability of dropping a packet, the paper recommends using a weighted
average of the queue size, rather than the queue size in any given moment.

Using a weighted average of the queue size makes the algorithm more resilient to a sudden
burst of traffic. If a new flow randomly pops onto the network throws a bunch of packets
onto the queue temporarily, that doesn't necessarily mean that congestion is happening.
We only want to increase the probability of a drop if a standing queue is building--meaning
that the queue is high for a period of time.

The way you compute the weighted average is as follows:

$$weighted\_average  = w_q * current\_queue\_size + (1 - w_q) * weighted\_average$$

$$w_q$$ is some parameter to the RED queue, and is a measure of how much you want to
weigh recent measurements of the queue size relative to older measurements. This gives
the $$weighted\_average$$ the property that older measurements of the queue size
decay exponentially. We do this update every time we enqueue a new packet to figure out whether
it should be dropped or not.

## A Bug

Alright, that all seems fine--but when we actually wrote up our implementation, we started
noticing some odd behavior. Notably, on some experiments, the congestion window would just
collapse to nothing and never recover. We'd see graphs like this for the congestion window:

![congestion_collapse]({{ "/assets/congestion_collapse.png" | absolute_url }})

And the link queue size graph would look like:

![congestion_link_queue]({{ "/assets/congestion_collapse_link_queue.png" | absolute_url }})

It took us a while to figure out what was going on here, but we ultimately figured out that
there's a negative spiral that happens:

RED begins dropping packets probabilistically because a queue starts filling up. The TCP
senders begin dropping their windows, resulting in fewer packets being sent through the
link. Since the window size is now small, even though the link queue is empty, the weighted
average is still very high. This results in packets continuing to be dropped, resulting
in TCP senders dropping their senders. Since fewer packets are arriving at the link now,
there are fewer opportunities for the weighted average of the queue size to be updated.

## A Solution!

The fundamental problem here is that if packets aren't arriving at the queue, and the queue
is empty, there is nothing that happens to the weighted average.

The solution to this is to behave differently when packets arrive at an empty queue. Rather
than simply decaying the existing average by `wq`, instead, decay it by the *amount of time
that the queue has been empty*.

$$weighted\_average = (1-w_q)^{(current\_time - time\_queue\_become\_empty)/average\_transmission\_time} * weighted\_average$$

Here, `average_transmission_time` is the typical amount of time between packet enqueues. This
way, if packets don't arrive at the queue for a little while, that gets taken into account
in the weighted average calculation.

This problem and solution was laid out in the paper, but it was cool to have discovered this
on our own.

# Next up

While RED in our examples performs much better than droptail, it does have some weaknesses--for instance, RED requires some tuning to
work effectively. It has a handful of parameters, including the $$w_q$$ term that we discussed earlier,
and there isn't a clear science for figuring out the right values for these parameters. There
are other AQM approaches that are more commonly used today, like [CoDel](https://en.wikipedia.org/wiki/CoDel).

In the next couple posts, I'm going to return to solving congestion at the sender level and
discuss some of the work that I've been doing around using reinforcement learning to help
senders find the right congestion window.

Thanks for reading!

*Big thanks to Venkatesh Srinivas for the help implementing this, and for the introduction to [queueing theory](https://en.wikipedia.org/wiki/Queueing_theory)!*
