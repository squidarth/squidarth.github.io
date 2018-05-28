---
layout: post
title:  "Using Linux Raw Sockets"
date:   2018-05-23 14:04:38 -0400
categories: networking systems rc
---

In an effort to learn how TCP/IP works, I decided to start playing around with
a low-level TCP/IP library, [smoltcp](https://github.com/m-labs/smoltcp). Since it is
a standalone networking stack and needs to send/receive raw packets, it needs to be
run as the root user.

To skip ahead, the [github page](https://github.com/m-labs/smoltcp) provides some shell
commands that can be used to allow programs using smoltcp to run as non-root users. Without
further ado, here they are:

To create a "tap interface" (more on this later):

```bash
sudo ip tuntap add name tap0 mode tap user $USER
sudo ip link set tap0 up
sudo ip addr add 192.168.69.100/24 dev tap0 # 192.168.69.100/24 this is a private IP address that refers to the range 192.168.69.*
sudo ip -6 addr add fe80::100/64 dev tap0
sudo ip -6 addr add fdaa::100/64 dev tap0
sudo ip -6 route add fe80::/64 dev tap0
sudo ip -6 route add fdaa::/64 dev tap0
```

And to connect to the internet:

```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.69.0/24 -j MASQUERADE
sudo sysctl net.ipv4.ip_forward=1
```

In this post I'll be digging into what exactly all these commands do. While the library
I'm working with is written in Rust, I'll use examples using C.

# Why does it mean to access a raw socket?

Sockets are the Linux way for computers to talk to the internet.
The [`socket` system call](http://man7.org/linux/man-pages/man7/socket.7.html)
creates a file descriptor that can be written to and read from. Writing to the
socket sends data to the remote address, while reading from the socket file
descriptor reads data sent from the remote address.

There are two main types of sockets, streaming sockets, and datagram sockets.
I won't get into the details of these now, but streaming sockets are for applications
that use TCP, while datagram sockets are for applications that use UDP. These are both
transport-level protocols that follow the network-level IP protocol.

If you are interested in writing your own implementations of one of these protocols,
or need to use a different transport-layer protocol, you'll need to use _raw_ sockets.
Raw sockets operate at the network OSI level, which means that transport-level headers
such as TCP or UDP headers will not be automatically decoded. If you are implementing a
a transport-level protocol, you'll have to write code to decode and encode the transport-level
headers in your application.

# Raw Sockets and Security

An important thing worth noting about this is that there is important security-related
information stored in TCP and UDP headers:

![TCP Packet Diagram]({{ "/assets/TCP_Protocol_Diagram.png" | absolute_url }})

For instance, the destination port is stored in the TCP headers. This means that packets
read from raw sockets don't have any notion of "port".

To step back a little bit, an application using TCP or UDP must, when opening up a STREAM or DATAGRAM socket, must declare a port to receive data on.  When the application reads data from that
socket, they  will only see data that was sent to that particular port.

Now, with raw sockets, because network-level IP packets do not have a notion of "port", all
packets coming in over the server's network device can be read. Applications that open raw sockets
have to do the work of filtering out packets that are not relevant themselves, by parsing the TCP
headers. The security implications of this are pretty serious--it means that applications with
a raw socket open can read any inbound network packets, including those headed to other applications
running on the system, which may or may not be run as the same unix user as the application with
the raw socket open.

To prevent this from happening, Linux requires that any program that accesses raw sockets be
run as root. Actually running network programs as root is dangerous, especially for a sufficiently
complicated program like a TCP implementation.

In this post I'll cover a couple different strategies for circumventing this.


# Attempting to read from a raw socket

To see what happens when we read data from sockets, let's use this C program that
simply sniffs packets entering the system and prints out some information about
the packet as an example:

{% highlight c %}
// raw_sock.c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<netinet/ip.h>
#include<sys/socket.h>
#include<arpa/inet.h>

int main() {
    // Structs that contain source IP addresses
    struct sockaddr_in source_socket_address, dest_socket_address;

    int packet_size;

    // Allocate string buffer to hold incoming packet data
    unsigned char *buffer = (unsigned char *)malloc(65536);
    // Open the raw socket
    int sock = socket (PF_INET, SOCK_RAW, IPPROTO_TCP);
    if(sock == -1)
    {
        //socket creation failed, may be because of non-root privileges
        perror("Failed to create socket");
        exit(1);
    }
    while(1) {
      // recvfrom is used to read data from a socket
      packet_size = recvfrom(sock , buffer , 65536 , 0 , NULL, NULL);
      if (packet_size == -1) {
        printf("Failed to get packets\n");
        return 1;
      }

      struct iphdr *ip_packet = (struct iphdr *)buffer;

      memset(&source_socket_address, 0, sizeof(source_socket_address));
      source_socket_address.sin_addr.s_addr = ip_packet->saddr;
      memset(&dest_socket_address, 0, sizeof(dest_socket_address));
      dest_socket_address.sin_addr.s_addr = ip_packet->daddr;

      printf("Incoming Packet: \n");
      printf("Packet Size (bytes): %d\n",ntohs(ip_packet->tot_len));
      printf("Source Address: %s\n", (char *)inet_ntoa(source_socket_address.sin_addr));
      printf("Destination Address: %s\n", (char *)inet_ntoa(dest_socket_address.sin_addr));
      printf("Identification: %d\n\n", ntohs(ip_packet->id));
    }

    return 0;
}
{% endhighlight %}

Compiling this program with:
```bash
$ gcc raw_sock.c
```

And then running it with:
```bash
$ ./a.out
```

yields the following error:

```
Failed to create socket: Operation not permitted
```

# Options

The most basic thing that you can do to run the program
is to run the program as root:

```bash
$ sudo ./a.out
New Packet:
Packet Size (bytes): 40
Source Address: 10.0.2.2
Destination Address: 10.0.2.15
Identification 2541

New Packet:
...
```

This is very clearly not ideal. With root access, programs can
do some serious damage to the system, and with a sufficiently
complicated network program, the chances of something going wrong,
like a code injection bug go up.

## Linux Capabilities

Thankfully, programs don't have to be run with full root access in order
to be able to use raw sockets. Linux has a feature called [_capabilities_](http://man7.org/linux/man-pages/man7/capabilities.7.html).
Linux capabilities allow _some_ features previously limited to processes running as root (with `sudo`)
to be used by processes that are unprivileged. The very handy `CAP_NET_RAW` capability can be used
to open raw sockets.

Capabilities are applied on a per-file basis with the `setcap` command. This example applies the
`CAP_NET_RAW` and `CAP_NET_ADMIN` capabilities to the `a.out` binary. Once these capabilities
have been set on the file, non-root users will be able to run these programs.

```bash
$ sudo setcap cap_net_admin,cap_net_raw=eip a.out
```

You can verify that the capabilities were added by running:

```bash
$ sudo getcap a.out
a.out = cap_net_admin,cap_net_raw+eip
```

Setting capabilities on executables allows you to run these programs
without having to run the risk of these being run as root.

There are two downsides:

1. This doesn't work on other operating systems like macOS
2. It allows any user in the system to run the program with that capability. In some setups, this could be problematic.


See these two great blog posts for more details on Linux capabilities in this context.

* [Sniffing with Wireshark as a Non-Root User](http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/)
* [File Capabilities in Linux](http://www.andy-pearce.com/blog/posts/2013/Mar/file-capabilities-in-linux/)


## Virtual Network Interfaces

Another way to run this program without having to resort to using `root` is by creating a virtual
network interface, the approach suggested by smoltcp. A virtual network interface is a pure-software
construct in Linux that creates a network device that can accessed similarly to physical network
devices. Importantly, traffic can be forwarded from the physical device to the virtual device
and vice-versa. On top of that, virtual network devices can be granted to particular unix users
on a server.

The strategy then to allow a non-root unix user to open raw sockets on the machine is to create
a virtual network device that is set up to forward traffic to/from the physical network device
and granted to a particular non-root unix user. With that, this unix user will be able to open
raw sockets on the virtual device, and therefore be able to read packets coming in/out of the
physical device.

Let's unpack the lines required to make this happen:

```
sudo ip tuntap add name tap0 mode tap user $USER
```

This command uses the `tuntap` command in the `ip` tool to create a
a `tap` mode virtual interface. TAP mode interfaces operate 

## Conclusion
