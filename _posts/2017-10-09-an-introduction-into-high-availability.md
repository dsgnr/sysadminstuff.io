---
layout: post
title: An Introduction into High Availability
categories: Misc
permalink: /:categories/:title
post_description: Gone are the days of lengthy downtime and unreliable services. But what is high availability?
cover: "/assets/images/post-thumbnails/introduction-into-high-availability.svg"
cover_color: "#f87e40"
author: dan_hand
---

Introduction
In recent years the hosting landscape has changed dramatically, gone are the days of lengthy downtime and unreliable services. This is mostly down to the rise of new technologies enabling high available / fault tolerant systems. But what is high availability?

The Wikipedia definition is as follows; [https://en.wikipedia.org/wiki/High_availability](https://en.wikipedia.org/wiki/High_availability):

> High availability is a characteristic of a system, which aims to ensure an agreed level of operational performance, usually uptime, for a higher than normal period.

In the technology world HA can take many forms. Some of the most common are:

* Networking
* Applications
* Services
* Storage
…and many more.

### So, how do you achieve HA?
The simple answer is you remove any Single Point Of Failure (SPOF) within your setup that could leave to service outage, and have a reliable switchover from the failed entity. Consider the following setup;

<figure><img src="/assets/images/intro-into-high-availability-non-ha.svg" alt="Introduction into high availability non ha"></figure>

Should the router fail in this scenario – what happens to the end user PC’s? They lose connectivity and manual intervention is required to restore service. But how would you fix this?

In this case, it would look something like this:

<figure><img src="/assets/images/intro-into-high-availability-ha.svg" alt="Introduction into high availability ha"></figure>

We have added a second router, and combined them using a floating IP and a protocol called VRRP (Virtual Router Redundancy Protocol – More on this to come in another post).

This means should the active router fail, the remaining router will pick up the slack and service will remain available. But this setup is not yet completely HA – why you ask? Because we have the middle switch as another SPOF.

### Active – Passive?
In the HA world common phrases you will hear are Active – Passive, Active – Active, Master – Slave and Master – Master. These refer to the operational state of the entities providing the HA service. For example in my example above, the routers using VRRP would be in an Active – Passive setup. One router would be performing all of the network routing, while the other waits in standby.

### What are the disadvantages of HA?
The main disadvantages of any high availability setup are:

* Cost – For the most part you have to purchase double the amount of equipment
* Complexity – The setups require a strong technical knowledge and a large amount of complexity to setup correctly
* Efficiency – Much like with a RAID array, you lose a certain percentage of your capacity through redundancy. The same applies for everything else, in my setup above you would have to power two routers all the time

### How do I get HA in my setup?
How long is a peice of string? This depends on what you run and how you run it, but we will be covering many ways you can get HA for common services in future blog posts!
