---
layout: post
title: "Web SevChat"
tagline: "connecting Java to Erlang just because we can"
description: "SevChat web chat connects Java with Erlang through OTP. Web client uses Stripes framework and postreSQL.  Chat Server is written in Erlang."
author: Vsevolod Geraskin
category: projects
tags: [java, erlang, stripes, javascript, ajax, otp, postgresql]
excerpt: "In my distributed systems BCIT option, I learned Java Stripes web framework and Erlang functional language.  Naturally, I was curious about ways to connect the two together."
---
{% include JB/setup %}

<img class="float-right" width="480pt" src="/assets/post_images/sevchat1.png" alt="System Diagram of SevChat" />

#### Problem Statement
In my distributed systems BCIT option, I learned Java Stripes web framework and Erlang functional language.  Naturally, I was curious about ways to connect the two together.

#### Project Description

The chat application uses the following components:

- to handle chat messages and send them to users, Erlang sevchat_server,
- to send and receive messages to/from Erlang server, java classes with OTP Erlang library,
- to handle clients webpage functionality, stripes framework and javascript/AJAX,
- to create and store clients logins, Erlang node settings, etc; postgreSQL database.

The user are able to do the following:

- register themselves with their username, password, and name,
- log in with their username and password,
- view all connected users,
- chat with all connected users,
- and send private messages to selected users.

Finally, all messages are logged in PostgreSQL database.

#### Project Outcomes
The system works and is available on [github](https://github.com/past5/sev-chat).  Also, check out Erlang [server source](https://github.com/past5/sev-chat/blob/master/erlang/sevchat_server.erl) and 
[client source](https://github.com/past5/sev-chat/blob/master/erlang/sevchat_client.erl).  The full report is available 
[here](/assets/post_docs/SevChatGeraskin.pdf).
