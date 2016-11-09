---
layout: post
section-type: post
title: Deployment in micro-services era
category: "Shipping software"
---

A solution for deploying a system composed of multiple services requires more than set of features provided by typical PaaS. Such platforms focus usually on deploying a single HTTP service with its backing services like cache, database or messaging broker. They do usually great job. However, most of the projects will end up having more than one service. It may be a background image processing worker, a mailer or separate service written in Elixir for receiving bursts of events from IoT devices.

I think it is very important to find own balance between shiny bells and the bare minimum required to bring an application to the user. I can't imagine working on the deployment pipeline for months just to ship a couple of micro-services! On the other hand, developers should not be left without tools empowering them to deploy often and with confidence. The worst thing can happen is a developer, who has just implemented a long-awaited feature, being scared to deploy, because he has no clue, whether the system is still going to work after the deployment is over.

What traits should a deployment pipeline for a young startup have? Let's go through the list!

- **fast**. It seems obvious and because it's obvious I believe it's undervalued. "*It's just 10 minutes more, cmon*" - you can hear. The truth is that if your deployment cycle net (and gross) time is unnecessarily long then the developer will refrain from integrating their feature branch often. It will increase the cost of merging and synchronization

- **declarative**. I should be able to check in the structure of my system and its configuration into version control. Developers know how to use version control. It's natural for them. It's the easiest thing to use for them. I don't see any reason to use graphical user interface to click out my system. *Why on earth I have to use THE GUI? How to revert my last change?* Declarative deployment pipeline makes me sure that even if I screw up badly, I can rebuild everything automatically, given, I can restore the state of the databases.

- **extensible**. I don't want to implement everything today, but I want to be able to add more layers and features later: monitoring, centralized logging, ssh key management or custom RabbitMQ server.

- **system-oriented** I don't want to ship one service. I want to deliver the whole product to the user. This means that even I am currently deploying a single application, the whole system should be taken into the consideration. If some variables change because of my deployment, I want also update all dependencies, and more importantly - in the correct order! I don't want to think about the details **every time**.

This list is not exhaustive. Everyone has own priorities and could add even more to this list. To me, this is essential, when talking about the deployment pipeline, which should be a tool empowering the developer to deliver.