---
title: "Red Teaming Power Users Pt. 1 - Terminals Programs"
last_modified_at: 2016-03-09T16:20:02-05:00
categories:
  - Red Team
  - Hacking
  - Enumeration
tags:
  - PuTTY
  - MTPuTTY
  - MobaXTerm
---

While performing Red Teams, you may occasionally find yourself on a Power Users box, with no obvious ways to Escalate privileges. You could normally start scanning the network, to try to identify boxes where your current user context DOES have administrator privilege, and move laterally to there. But what about if you want to limit the amount of network traffic you're generating, in order to avoid detection by the Blue Team? In this series of posts, I'm going to discuss different technologies in use by Power Users and how you can use them for different attacks.

In part 1 I'm going to discuss common Terminal applications that you could use for Enumeration, Lateral Movement, and Privilege Escalation. Specifically we're going to discuss the following applications:
* PuTTY
* MTPuTTY
* MobaXTerm
* KiTTY

### PuTTY
[PuTTY](https://www.putty.org/) is a pretty standard SSH and telnet client. It's one of the most popular clients out there and will likely be the one you see on your assessments. Like the other applications in this post, it has the ability to save user sessions for easier access. Each saved session can contain multiple configuration options which will be dependant on what your user decides to save. From an enumeration standpoint, we're mainly going to be interested in uncovering the following items:
* Username
* Hostname/Port
* Credential Information (Password/Keys)

The good news for us is, that it's super easy to gain access to all the information we need. PuTTY while being easy to use, stores all of it's information in cleartext in the registry. Each session is stored under a registry key located at `HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\`

### MTPuTTY



### MobaXTerm



### KiTTY
