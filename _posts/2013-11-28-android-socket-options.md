---
layout: post
title: "Android Socket Options Tester"
tagline: "exploring properties of TCP/IP connection specific to Android over WIFI (exciting stuff!)"
description: "This paper explores various TCP/IP socket options exposed by Android API and their effect on TCP/IP connection over 802.11 WIFI network."
author: Vsevolod Geraskin
category: projects 
tags: [android, java, c, linux, tcp/ip, 802.11]
excerpt: "What do all those TCP options do and when to use them?"
---
{% include JB/setup %}

#### Problem Statement
What do all those TCP options do and when to use them?

#### Project Description 
TCP Socket Android Performance Tester is an Android application I wrote while studying at BCIT. The goal of this project was to understand Android TCP/IP socket options and analyze potential
scenarios when each option should be used. For some of the socket options, we also evaluated the impact on TCP connection. For the above purposes, each TCP socket option was independently set and
tested in Android client / Linux server environment on 802.11 wireless network. At the same time, TCP packets were captured and analyzed in Wireshark.

Software used for testing:

- Fedora Release 15 32-bit
- Multiprocess socket server written in C
- Wireshark Packet Capture Analyzing Tool
- Android 4.2.2
- Android socket client written in Java

Hardware used:

- Samsung Galaxy Tab 3 8.0
- Compaq Laptop Intel Pentium Dual CPU, 1.86 GHz x 2, 3 Gigs of RAM

Both server and client were connected to 802.11n/g/b wireless network provided by Dlink DIR-615 router.

#### Project Outcomes
Linux people who hack C will not find anything new here.   Still, its nice for Android to expose some fairly low level socket API.  Stuff like finding optimal buffers for uploading files is
a potential challenge you might face when building scalable client/server systems.  If you are curious when to use this, in all unlikelihood, here's some bedtime 
[reading material](/assets/post_docs/SocketOptionsGeraskin.pdf)...