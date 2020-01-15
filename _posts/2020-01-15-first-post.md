---
title: "Forerunner: Commanding your Covenant"
categories:
  - Infosec
  - Tools
tags:
  - Covenant
  - Automation
  - Red Team
---

As Blue Teams get quicker and more effective at discovering and remediating intrusions, Red Teams need to move quicker and quicker in order to successfully achieve their objectives. A big part in accomplishing this is automating common TTPs.

It happens constantly, you gain access to a box, you do your initial checks (am I in scope, am I in a sandbox, am I admin, etc etc). It may have become second nature after repeatedly doing it, but that time adds up through the lifetime of an operation. Even worse, is when you send out your phishing e-mail and you wait for a return. You step away to spend some time with family, grab some lunch, or just get some time away from the computer and your agents comes back and is just calling out for commands, generating traffic that could eventually trigger an alert.
Preventing these situations, and making your workflow more efficient are the reasons why it's important for C2 platforms and Red Teamers to have ways to automate their common workflows. Cobalt Strike does this well with its Aggressor language, allowing code execution upon initial beacons and adding custom commands to your UI. But what about other C2 providers?

In this Blog Post I'm going to discuss a tool I've been working on named Forerunner. This is a tool that makes extensive use of the Covenant API in order to allow operators to hook into Grunt Events, and perform actions based on the event that's returned.
