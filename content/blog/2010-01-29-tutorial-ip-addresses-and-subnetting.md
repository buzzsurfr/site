---
title: "Tutorial: IP Addresses and Subnetting"
date: 2010-01-29T22:21:16-05:00
draft: false
categories: ["Deep Dive"]
tags: ["binary", "Dotted Decimal Notation", "Host ID", "IP", "IP Address", "Network ID", "Subnet", "Subnetwork"]
description: "An intuitive approach to understanding the basics behind the format of an IP address, how a subnet works, and how to make a subnet work for you...."
image: ""
---

When you connect your computer to the internet, your computer is given an *Internet Protocol* (IP) address.  If you check your connection settings, you will commonly see two similar values; one is called a *subnet* and the other a *gateway*.  Have you ever wondered what these three actually mean and what they do?

An IP address is like your address at your home.  Your home address tells other people where to find you when you're home, and your IP address tells other computers where to find your computer.  The most common form of an IP address is called *dotted decimal notation*, which has four sets of numbers, each ranging from 0 to 255, separated by a period.  192.168.0.1 is a common example, and I'll use it throughout this tutorial, so write it down!

Why 0 to 255?  It is the range of an *octet*, or 8-bit number.  As you've heard before, computers speak in binary, which is just 0's and 1's, on and off, etc.  Each bit can have a 0 or a 1, so the range 0 to 255 is really 00000000 to 11111111 in binary.  There are four octals in an IP address, which is a fancy way of saying an IP address is a 32-bit number.

Which of these numbers is easiest to understand?

	
1. 192.168.0.1 - Dotted Decimal Notation using octals
	
2. 11000000101010000000000000000001 - Binary
	
3. 11000000.10101000.00000000.00000001 - Dotted Binary
	
4. 3232235521 - Decimal Notation

They are all the same!  But #1 has proven itself to be the easiest to use, especially when you have subnetworks.

Subnetworks?  You mean there's more than one???  The reason relates to divide-and-conquer.  One person trying to manage 4294967296 computers would be tasking, to say the least.  Hence we can use subnetworks, or *subnets*, so that one computer is not responsible for the whole network.  With the hardware and software out today, it's easy for a device to manage it's own subnet.  But which one are you using?  How big of a subnet are you in?

The answer is right there in your subnet, sometimes called a subnet mask.  This tells us what subnet we are a part of, and how big the subnet is.  To make things easier, there are three defined subnets:

	
- Class A - 255.0.0.0
	
- Class B - 255.255.0.0
	
- Class C - 255.255.255.0

For our example, let's say that our IP address, 192.168.0.1, is in a Class C subnet, 255.255.255.0.  This means that 192.168.0.1 is in a subnet with 256 addresses ranging from 192.168.0.0 to 192.168.0.255.  ("How I did that" is coming up...)

An IP address is actually a combination of two IDs, a network ID and a host ID.  Notice in the range above, the 192.168.0 stayed constant while the last octet changed.  Simply put, the constant 192.168.0 is the network ID and the last octet is the host ID.  This is important so your computer knows what subnet you are on.

By now you've noticed that our subnet mask, 255.255.255.0 has no digits in common with our IP or our range of IPs--so why is it important?  Our subnet mask is the dividing line between network and host!  Let's look at our IP and subnet mask:

```
IP:     192.168.  0.  1
Subnet: 255.255.255.  0

```

That vaguely shows me that the first three octets are the network, but you've seen subnets that don't have 255 or 0 in all of the slots.

What does binary show us?

```
IP:     11000000.10101000.00000000.00000001
Subnet: 11111111.11111111.11111111.00000000

```

There it is!  Every bit in the subnet that's a 1 makes that bit in the IP part of the network.  Every bit in the subnet that's a 0 makes that bit in the IP part of the host.

Let's take a look at a different subnet mask: 255.255.255.128.  This isn't one of our three "class" subnets, and thus it is called a *classless* subnet.  Again, in binary:

```
IP:     11000000.10101000.00000000.00000001
Subnet: 11111111.11111111.11111111.10000000

```

Not a big change, but what does that do to our network?  First, you now only have 2^7, or 128, hosts in your subnet instead of 256.  And now your network ID has changed...

With our new subnet, is 192.168.0.127 in our subnet?

```
Subnet: 11111111.11111111.11111111.10000000
IP:     11000000.10101000.00000000.00000001 <-- 192.168.0.1
IP:     11000000.10101000.00000000.01111111 <-- 192.168.0.127
        ←          Network         →← Host→
```

Both IPs have the same Network ID, thus they are in the same subnet.

What about 192.168.0.128?

```
Subnet: 11111111.11111111.11111111.10000000
IP:     11000000.10101000.00000000.00000001 <-- 192.168.0.1
IP:     11000000.10101000.00000000.10000000 <-- 192.168.0.128
        ←          Network         →← Host→
```

These have different Network IDs, so they are in different subnets.

What if we go back to a Class C subnet?

```
Subnet: 11111111.11111111.11111111.00000000
IP:     11000000.10101000.00000000.00000001 <-- 192.168.0.1
IP:     11000000.10101000.00000000.10000000 <-- 192.168.0.128
        ←          Network        →← Host →

```

Now they are in the same subnet!

Before we noted that dotted decimal notation is preferred for writing IP addresses because it's the easiest to remember.  If you have a Class C subnet and are responsible for assigning IP addresses to computers, you know that [in our example] the first three octets will always be 192.168.0!  Much easier!

There's also a shorthand for writing subnets.  Just put a slash (" / ") after the IP address, and then write how many 1's are in your binary form of subnet.  For example, with a Class C subnet our shorthand would read 192.168.0.1/24, and our classless example would be 192.168.0.1/25.

In your network settings, there's also a *gateway*.  How does it relate?  The gateway is an IP address INSIDE your subnet which connects you to the OUTSIDE of your subnet.  If you try to send a packet to a computer and it's IP address isn't in your subnet, then your computer sends it to your gateway and the gateway passes the message on.

I know how you feel.  When I learned about subnetting, the only thing I could think of is, "how will this be useful?"  I then went into work and looked at my company's IP address, subnet, and gateway (I changed the company IP to protect my company, sorry):

```
IP:     192.168. 25.174
Subnet: 255.255.255.252
Gateway:192.168. 25.173
```

Back to binary:

```
IP:     11000000.10101000.00011001.10101110
Subnet: 11111111.11111111.11111111.11111100
Gateway:11000000.10101000.00011001.10101101

```

Let's look at the subnet first.  There are only two bits for hosts IDs (and 00 and 11 are reserved), which leaves exactly TWO IP addresses in this subnet.  This means that my computer is on a network BY ITSELF, and the other is used as the gateway!  This does not work for every case but when I have just one computer I don't want anyone else inside my subnet!

And my IP and subnet are easy to remember because I write it as 192.168.25.174/30!