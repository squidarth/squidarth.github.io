---
layout: post
title:  "What I learned attempting the TCP Reset attack"
date:   2020-05-03 14:04:38 -0600
authors: Sid Shanker
categories: article networking
---

Last week I read a [fantastic blog post](https://robertheaton.com/2020/04/27/how-does-a-tcp-reset-attack-work/) from [Robert Heaton](https://robertheaton.com/) about
TCP Reset attacks. I figured it would be a good exercise to try to get the author's attack code
working on Linux! I learned a ton about TCP and how to debug computer networks in the process.
In this post I give a high-level overview of what TCP Reset attacks are, and tell the story of
how I got the attack code working on Linux.

## What is a TCP Reset Attack?

For a fuller description of what a TCP Reset attack is, I'd highly encourage you
to read [Robert's blog post](https://robertheaton.com/2020/04/27/how-does-a-tcp-reset-attack-work/), but
here I give a high-level overview:

### Some Background on TCP

TCP is a networking protocol used for two parties on the internet to exchange information *reliably*.
In any connection between parties A and B, if A sends a packet of data to B, B will send an "acknowledgement"
packet indicating that B received that packet. If A does not see an acknowledgement for that packet from B,
following the rules of the TCP protocol, A will resend that data until it sees an acknowledgement.

An important detail about TCP is that all packets have ordered *sequence* numbers associated with them. The first
packet a sender sends is a random initial sequence number, and each subsequent packet will have a higher sequence number. When
the receiver acknowledges packets, they send back an *acknowledgment* number, indicating what the highest sequence number
they have received from the sender is.

The best way to understand this is to actually look at the TCP packets, using a tool called [Wireshark](https://www.wireshark.org/).

We start by using the `nc` program to start a TCP server and client:

![nc-1]({{ "/assets/nc-1.png" | absolute_url }})

We send the string "hello" from the client to the server. For this exchange we get the following packets (note that "hello" is 6
bytes because of the newline character):

![wireshark-tcp-intro]({{ "/assets/wireshark-tcp-intro.png" | absolute_url }})

Another important aspect of the TCP protocol is that it supports having different flags on TCP packets,
one of which is the "Reset" flag ("RST").
The idea behind this flag is that if a party on a TCP connection receives a TCP Reset packet from the other
party, it will immediately close the connection. This is intended to be used in cases where a computer receives a TCP
packet that it does not recognize -- in this case, the computer can send back a RST packet. This might happen
if a computer shuts down while a TCP connection is still open, and when it restarts receives packets intended
for that TCP connection.

### Now, for the attack!

Alright, so we know that parties A and B can each send packets with the reset flag set. However, it turns out,
that if you listen to the traffic that is happening on network, an attacker C can send a packet "pretending" to
be B, to A, with the RST flag set. A will then close its connection. The attacker has then caused the connection
to be terminated!

An important detail about this attack is that according to the TCP specification, in order for a TCP RST packet
to be accepted, the RST packet must have the *same sequence number that the recipient of the RST packet is expecting*.

Let's say A sends a single packet to B with a sequence number of 10, and an acknowledgement number of 12, that contains 5 bytes of data.
It will expect to receive an acknowledgment packet from B with a sequence number of 12, and an acknowledgemnet number of 15 (10 + 5).

B can send back that packet we just described, *or* it would be valid for B to send an RST packet with a sequence number of 12.
For a third-party C to cause the connection to die, it must send a packet to A, pretending to be from B, with a sequence number
of 12. This sequence number requirement is a security feature, as the attacker must be able to send exactly the sequence number
one of the parties received, which can be difficult (but still doable) if lots of data is being exchanged.

## What does TLS actually encrypt?

My first question when I read about this attack was -- well, isn't TLS (encrypted TCP, used for HTTPS connections) supposed
to defend against attackers "spoofing" packets or engaging in man-in-the-middle attacks like this?

The answer to this, upon further inspection, is that TCP header data, information like source ports and
destinations, is available for all to see! See this wireshark view of data from a TLS connection:

![wireshark-tcp-intro]({{ "/assets/tls-intro.png" | absolute_url }})

What TLS actually encrypts and can verify the identity of is the *contents* of the TCP packets.
Until you open the actual load of a packet sent via TLS, there is no way to verify that the sender
is who it claims to be. Anybody can send a TCP packet with a source IP/port set, and there's no
way of verifying that or not, absent looking at the contents of the packet in a TLS connection.

This is actually important to the functioning of the internet, as routers use knowledge of ports/ip
addresses to work properly (your WiFi router is probably using [NAT](https://en.wikipedia.org/wiki/Network_address_translation)).

Now, because TCP Reset attacks happen at the TCP header data level, there is no way
for the sender of the packet to be verified. So, TCP reset attacks can be executed, even
over TLS connections.

## Debugging the attack code

Now, returning to the premise at the beginning of the post, in the blog post I mentioned
at the beginning, there was some [attack code](https://github.com/robert/how-does-a-tcp-reset-attack-work/blob/77d06123b24a0b69f5ed829bcaeb3db4aa7add8e/main.py#L116-L119), that actually executes a TCP attack. I figured it would
be a good exercise to get it to work on Linux!

### The "Device" Name

The [script](https://github.com/robert/how-does-a-tcp-reset-attack-work/blob/77d06123b24a0b69f5ed829bcaeb3db4aa7add8e/main.py)
uses the [scapy](https://scapy.net/) library to sniff for packets, and for cases where the client sends a packet to the server,
sends a spoofed TCP packet with the RST flag back to the client.

I ran the script:

```
$ sudo venv/bin/python main.py 
```

and immediately got the error:

```
OSError: [Errno 19] No such device
```

A quick Google search landed me on this [Stackoverflow](https://stackoverflow.com/questions/43556566/socket-error-errno-19-no-such-device-scapy-python) page -- it turns out that Linux and MacOS name their loopback (localhost) devices differently. While
on Linux, the device is named `lo`, on MacOS, by default, it is `lo0`.

Changing this in the script immediately fixed the error.

### The script seems to work, but the connection is still open!

Alright, so now, with a netcat session open as follows:

![nc-1]({{ "/assets/nc-1.png" | absolute_url }})

In a separate window, I ran the attack script:

```
 $ sudo venv/bin/python main.py
Starting sniff...
```

After sending another message via netcat, the attack script
claimed to have intercepted the packet and sent a RST packet!

```
 $ sudo venv/bin/python main.py
Starting sniff...
...
Grabbed packet src_ip=127.0.0.1 dst_ip=127.0.0.1 src_port=33362 dst_port=8000 seq=1967401732 ack=1844274764
jitter == 0, this RST packet should close the connection
```

However, the TCP connection was still open!

### Let's take another look at Wireshark

At this point, there are a lot of things that could be going wrong.
At the packet level, it's possible that the script wasn't _actually_ sending
a packet. Maybe the packet was misconfigured? Maybe Linux and MacOS have implementations
that handle RST packets differently.

While the last hypothesis would be hard to test without doing a lot of research, with
our handy friend Wireshark, we could actually take a look at the packet we're getting
back!

So I ran the sniffer script again, and the packet did show up in Wireshark!

![rst-1]({{ "/assets/rst-1.png" | absolute_url }})

So it does look like a packet is being sent, the sequence numbers seem to
be correct, and the ports seem to be correct! So what's going on here?

One small thing you'll note is that the window size of the RST packet is different to
what we see on the other TCP packets in that wireshark dump.

#### Does the window size matter?

The next thing I tried was changing the window size to match up with the
other packets:

![rst-2]({{ "/assets/rst-2.png" | absolute_url }})

No cigar.

### Trying a new tool

It looks like the packet is configured correctly. So what's happening here?
We've ruled out the hypotheses that the script is not sending a packet, or that the
packet is misconfigured. But what if MacOS and Linux handle this flag differently?

The next step here was trying to find a different way to construct and send a
packet. If we can get this attack working with a different method, we can rule out
the fact that RST packets work differently in MacOS or Linux.

I read a little bit about other ways of sending a spoofed RST packet to the client,
and found this handy tool called [netwox](http://www.cis.syr.edu/~wedu/Teaching/cis758/netw522/netwox-doc_html/html/examples.html)
that can be used to forge packets. [This presentation](http://www.cse.iitm.ac.in/~chester/courses/19e_ns/slides/3_TCPAttacks.pdf) has
a good example of a TCP attack.

So, I used the `netwox` tool after installing it, by running:

```
$  sudo netwox 40 -l 127.0.0.1 -m 127.0.0.1 -o 8000 -p 33760 -B -q 3545181336
```

Where 33760 is the port number of the client, and 3545181336 was the correct sequence number.
Lo and behold, it worked! The connection closed!

![working]({{ "/assets/netwox-working.gif" | absolute_url }})

### Spot the difference between these packets!

Alright, so we have an example of the working packet, and an
example of the "not working" packet. Let's see if you can spot the difference!

![spot-the-diff]({{ "/assets/spot-the-diff.png" | absolute_url }})

It's subtle. It turns out the ethernet destination address on the packets
is different.

![spot-the-diff-answer]({{ "/assets/spot-the-diff-answer.png" | absolute_url }})

So, at this point, we know there's no difference between how MacOS and Linux
handle the RST flag in TCP. However, `scapy` and `netwox` configure the packet
differently such that the destination MAC address is the "broadcast" address
on `scapy` and `00:00:00:00` with `netwox`.

At this point I *finally* decided to look at the `scapy` documentation (probably
should have done this earlier) and it turns out in the troubleshooting section,
they have a section about [just this](https://scapy.readthedocs.io/en/latest/troubleshooting.html#i-can-t-ping-127-0-0-1-scapy-does-not-work-with-127-0-0-1-or-on-the-loopback-interface). From the docs, they discuss how the
"loopback" interface is different to other network interfaces, and how link-level packets
sent on the loopback interface don't actually get disasembled/assembled (and therefore read by applications).

The solution that `scapy` offers is adding the line of code `conf.L3socket=L3RawSocket` to your file.
I tried this out, and the TCP Reset attack actually worked!

### What's the deal with this socket setting?

What's actually going on here, and why did the change that `scapy` suggests work? The answer is
in the details of the [`socket` syscall](http://man7.org/linux/man-pages/man2/socket.2.html).
Note that in this page, it mentions that sockets can listen on different "domains", or "protocol families".
By default, `scapy` uses the `PF_PACKET` protocol family when it opens a socket to send a packet. This is
a link-layer (Ethernet) protocol, which is why packets sent via `scapy` by default have the ethernet broadcast
address (`ff:ff:ff:ff`) as a destination. These packets, however, being link-layer packets, when addressed to the loopback address
don't get routed there correctly, even though they show up in wireshark, because of how the loopback interface
works in Linux.

Using the `PF_INET` protocol family, packets *do* make it to applications listening on the loopback interface,
because these packets are network-layer packets that do get routed appropriately by the OS to applications listening
on the loopback interface.

# Conclusion

While I probably should have dug into what exactly `scapy` was doing earlier, going through this exercise of debugging
*why* the TCP Reset attack wasn't working was really helpful and taught me a lot about computer networks
and how to debug them!

## Further Reading

* [How does a TCP Reset Attack Work?](https://robertheaton.com/2020/04/27/how-does-a-tcp-reset-attack-work/)
* [Socket man page](http://man7.org/linux/man-pages/man2/socket.2.html)
* [Inappropriate TCP Resets Considered Harmful](https://tools.ietf.org/html/rfc3360)
* [The TPC RFC](https://tools.ietf.org/html/rfc793)
