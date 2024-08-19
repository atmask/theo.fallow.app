---
title: 'Understanding CIDR Blocks and IPv4 Addressing'
tags: ["networking", "explainer", "cloud", "IP"]
ShowToc: true
date: '2024-08-09T20:01:16-04:00'
draft: false
---
# Big Idea

The goal of this post will be to give an overview of IPv4 addresses. My aim is to do this incrementally by first covering the anatomy of an IPv4 address in its base 10 and binary representations. Second, I will look at CIDR block subnetting and subnet masks. Thirdly, I will append some helpful formulas for working with IP addresses.

# Background & History

Before diving into the anatomy of IPv4 let's turn back the page. In 1981, RFC 791 was published. This document outlined the _Internet Protocol_ and the IPv4 addressing scheme that would be used to uniquely identify and locate machines on the Internet.

The ability to uniquely identify devices on a network is incredibly important. This ability is how we route packages from one machine to another across a network. On a network, a given IP must be uniquely assigned to a single device for seamless communication. The concept of the IP address that we will explore today is one part of a larger layered model (known as the OSI model) that facilitates the transfer of data over a network.

Engineers at the time that RCF 791 was published did not realize the magnitude of devices that would one day be connected to the Internet. This has become a problem in recent times as there are a fixed number of globally unique IPv4 addresses. As the number of devices has grown to surpass the number of available IPs, engineers have had to apply solutions such as Network Address Translation (NAT) to use IPs more effectively. This exhaustion of IPv4 addresses has also led to a second standard in IP addressing known as IPv6 which is becoming increasingly common. IPv6 functions in a similar fashion to IPv4 but has roughly 3.4×10^38 addresses compared to IPv4's approximately 4x10^9 addresses. 


# Anatomy of IPv4

An IPv4 address that you see may look like the following: `192.168.0.2`. This address represents 32 bits in a base 10 format. Those 32 bits are divided into 4 groups known as **octets**. Each octet contains 8 bits. The four octets correspond to the four '.' separated parts of the IP address in our example. When working with IPv4, it is often easier to convert the base 10 represenation of the IP address into binary. This means that we could represent the above IP as follows:

<div style="text-align: center;" markdown="1">

`192.168.0.2`

`1100 0000 - 1010 1000 - 0000 0000 - 0000 0010`
</div>

The lowest value in an octet is 0. In binary, this would correspond to an octet of all 0 bits: `0000 0000`. The highest value in an octet is 255. This is the highest number that can be represented with 8 bits (2^8-1): `1111 1111`. We subtract 1 from 2^8 since 2^8 represents the total unique values that can be represented with 8 bits, however, 0 is one of those values that can be represented meaning the values range from 0-255 (i.e. 2^8-1).


# Subnetting IPv4

It is common, when working with IP addresses in the cloud, that you want to create groups of sequential IPs that you can dedicate to a certain use. Often when people talk about these groups you will hear things like "IP space", "CIDR block", "subnet", or "address range". The key idea is that you want to carve out a broad range of IPs and divide those IPs in an effective way. This always involves balancing two variables: the number of subnets you want to create and the number of devices in each subnet. This segues into the next idea which is that each 32-bit IPv4 address is divided into two parts. A network portion and a host portion. 

Let's say we have a network consisting of all of the IP addresses between `10.0.0.0` to `10.255.255.255`. In that range of IPs, we see that the leading octet always has a value of 10 on the network (ex. `10.0.0.1`, `10.100.29.0`). This is because the first octet, that is the first 8 bits of this IP address, are fixed and represent the **network portion** of the IP address. The following 3 octets can change in value and represent the **host portion** of the IP. Let's look at this in binary:
<div style="text-align: center;" markdown="1">

10.0.0.0

**0000 1010** - 0000 0000 - 0000 0000 - 0000 0000 

(The leading 8 bits in bold are fixed and make up the network protion of the address)
</div>

In this example, the host portion allows 16,777,216 hosts. It is very likely that we don't want to dedicate all of those addresses to a single project. Instead, we want to partition out a section of IPs for our use-case. 

### CIDR Notation

When it comes to dividing an IP into the network portion and the host portion, the network portion is always represented by some leading number of bits. How many leading bits are fixed is often up to you, the developer, to decide. This means that we need a standard way to tell others how many leading bits are fixed and represent the network portion of the address. To solve this problem we will introduce CIDR notation. CIDR notation stands for Classless Inter-domain Routing and is a standard way to tell others how many leading bits make up the network portion of an address. The name, classless inter-domain routing is a nod to an earlier and antiquated method of partitioning IP addresses known as class-based routing. 

Using CIDR notation you will append an `/x` to the end of your network IP. The value of `x` dictates how many leading bits dictate the network portion of the address. In the example above we would represent our address range of `10.0.0.0` to `10.255.255.255` as `10.0.0.0/8`.

The number of leading bits does not need to cleanly divide the octets. This means you can have CIDR blocks other than `/8`, `/16`, `/24`, or `/32`. You could also have a CIDR block like `/12`. When creating a CIDR block that splits and octet the binary notation becomes increasingly helpful. Take a look at the following:

10.0.0.0/12

**0000 1010 - 0000** 0000 - 0000 0000  - 0000 0000 

10.0.x.x -> 10.15.x.x


### Subnet Masks

Subnet masks are another common notation for indicating the network and host portions of an IPv4 address. Subnet masks resemble an IP address. They are 32 bits divided into 4 otects. Subnet masks work like a kind of filter against which the 32 bits of an IP address can be OR'ed. To make this concrete let's take the IP address `192.168.0.2` and the subnet mask `255.255.255.0` and find the host and network portions.

|Type |   Decimal     | Binary                                         |
|-----|---------------|------------------------------------------------|
|IPv4 | `192.168.0.2` |`1100 0000 - 1010 1000 - 0000 0000 - 0000 0010`|
|Subnet Mask| `255.255.255.0` | `1111 1111 - 1111 1111 - 1111 1111 - 0000 0000` |
| OR  | `192.168.0.0` |`1100 0000 - 1010 1000 - 0000 0000 - 0000 0000`| 

By OR-ing the bits of the IP with the bits of the subnet mask, we are left with the host network. There is a condition for subnet masks. A subnet mask is 32 bits, which is always a contiguous series of 1s followed by a contiguous series of 0s. This means that, when converted to binary, a subnet mask will have a first half that is all 1's and a second half that is all 0's.  

### Host Address and Broadcast Address

An important thing to know about subnets is that two IPs in any given subnet are always reserved. The first is the lowest IP in that subnet. The lowest IP in a subnet is the **network address**. The network address uniquely identifies the network and is not used for hosts in that network. The second reserved address is the **broadcast address**. The broadcast address is the highest/last IP address available in the network. Any message sent to the broadcast address of a network is sent to all machines on that network. 

# Useful Formulas and Notes

### Number of IPs in a CIDR Block
`2^(32 - CIDR block leading bits)`

### Number of Unique Address of x Bits
`2^x`

### Largest IP of x Bits
`2^(x - 1) # 0 is one of the available values so subtract 1`

### Resizing CIDR blocks
Every time you decrease the number of bits in the network portion of an address you add bits to the host portion. This means that more IPs are available.
For every bit that you decrease the network potion by, the number of available IPs in the host portion of the network number is doubled. 



