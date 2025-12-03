---
date: "2025-11-29"
title: "TILs"
pubDate: 29 November 2025
description: ...
draft: false
slug: tils
---

In this section, I will record any 'Today I Learned' (TIL) notes from the AI Accelerator that don't make it into one of my full blog posts.

### CIDR IP ranges

I've seen these littered around Terraform configs, but had never had the chance to look up how to interpret them. CIDR stands for Classless Inter-Domain Routing, and it is a way of allocating IP addresses when provisioning resources on a network. The first part describes an IP address, the first IP address in the range. The bit after the forward slash is how many bits are fixed in the prefix. In IPv4, there are 32 bits in total (four sets of eight bits, separated by dots).

- `0.0.0.0/0` denotes all IP addresses (the entire IPv4 range)
- `192.168.0.1/32` is a single address
- `10.0.1.0/24` is a range of 256 addresses  --> (`10.0.1.0` -- `10.0.1.255`)
- `10.0.0.0/16` is a range of ~65k addresses --> (`10.0.0.0` -- `10.0.255.255`)
- `10.0.0.0/8` is a range of ~16.7m addresses --> (`10.0.0.0` -- `10.255.255.255`)

The following ranges are reserved for private IPs by convention

- `10.0.0.0/8` (~16.7m - big businesses)
- `172.16.0.0/12` (~1m - small businesses)
- `192.168.0.0/16` (~65k - home)

In a typical Terraform config, you might see a VPC (virtual private cloud) defined with a CIDR block of `10.0.0.0/16` (65k addresses), and then subnets defined within that VPC with CIDR blocks of `10.0.1.0/24`, `10.0.2.0/24`, etc. (256 addresses each).  