---
layout: post
title: "Claimstream Middleware Client"
tagline: "enabling pharmacies to electronically submit claims to third party insurers"
description: "Claimstream secure client is written in C and allows secure transmission of third party Alberta Blue Cross and Medavie Blue Cross Claims"
author: Vsevolod Geraskin
category: projects
tags: [c, tls/ssl, security]
excerpt: "Pharmacies and Hospitals in British Columbia use secure electronic middleware to send claims to third-party Claimstream insurers, such as Alberta Blue Cross and Medavie Blue Cross.  The insurers made
several technological changes on the receiving end which immediately required a completely new system to enable insurance claim submissions by our clients."
---
{% include JB/setup %}

#### Problem Statement
Pharmacies and Hospitals in British Columbia use secure electronic middleware to send claims to third-party Claimstream insurers, such as Alberta Blue Cross and Medavie Blue Cross.  The insurers made
several technological changes on the receiving end which immediately required a completely new system to enable insurance claim submissions by our clients.

#### Project Description
Alberta/Medavie Blue Cross communication software is a secure client application that transmits Canadian Pharmacists Association claim standard data from Applied Robotics WinRx pharmacy system to Claimstream, 
and captures claim responses from Alberta Blue Cross and Medavie third-party private insurers.  The application is written in C and uses OpenSSL for handling certificates and encryption.

#### Project Outcomes
The application is currently deployed in production at several hundred client sites.  Due to simplicity of the solution, all the clients had to do is to download the application from the website without
having to install any components.  I developed the application in about a day, and the total effort from initiating the project to testing with insurers and deploying at clients sites took about week. 