---
layout: post
title: "Covert Backdoor"
tagline: "testing Linux systems for software firewall vulnerabilities"
description: "This project implements a complete covert application that allows a user to communicate with a disguised backdoor application on a compromised machine around the software firewall and issue commands remotely."
author: Vsevolod Geraskin
category: projects
tags: [c, tcp/ip, linux, http, security]
excerpt: "All companies face challenges of securing their electronic data.  Software firewalls often have vulnerabilities which allow exfiltration of sensitive data by malicious software.  During my time at BCIT,
we examined whether software firewall alone is sufficient to prevent and/or track loss of data if the target machine is compromised. "
---
{% include JB/setup %}

<img class="float-right" width="480pt" src="/assets/post_images/covert1.jpg" alt="Covert Backdoor Network Diagram" />

#### Problem Statement

All companies face challenges of securing their electronic data.  Software firewalls often have vulnerabilities which allow exfiltration of sensitive data by malicious software.  During my time at BCIT,
we examined whether software firewall alone is sufficient to prevent and/or track loss of data if the target machine is compromised. 

#### Project Description

This project implements a complete covert application that allows a user to communicate with a disguised backdoor application on a compromised machine around the software firewall and issue commands 
remotely.  The backdoor application will accept commands and execute them; the results of the command execution will be sent back to the remote client application. 

The system works as follows:

- Using libpcap at both the client and the compromised server, we bypass the software firewall.
- The packets will arrive to the covert collector on port 80 with certain parameters.
- The command packet will arrive to the backdoor on a random port with certain parameters.
- Backdoor has a separate thread that sends SYNs to the covert collector to let it know its ready to receive commands.
- The backdoor process is masked.
- The temporary files are shredded off the system after sending and the backdoor program shreds itself out upon receiving a certain command.
- Crafting custom TCP packets according to our covert channel design.
- Inotify is used to monitor a defined directory for new files, then send those files to the covert collector.

#### Project Outcomes

The covert collector and the backdoor application communicate through a covert channel that mimics HTTP protocol, thus possibly escaping notice by network admins. Furthermore, libpcap packet capture 
library allows our channel to bypass software firewalls. If anything, the project demonstrated that software firewalls alone will not prevent the loss of data from a compromised system.