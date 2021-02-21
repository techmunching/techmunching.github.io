---
layout:	post
title:	"Smart Observer Pattern in C++"
description: "Event dispatcher helps to communicate across different components in an architecture. This tutorials shared an advanced architecture for event dispatcher which handles life time of event in an multithreaded environment. Let's look into..."
date:	2019-10-13
categories: [ 'C++' ]
image: '0*5TpHrNqaemC7yh-U'
tags: [ featured ]
author: admin
---

  smarter pub-sub A better event dispatcher
  {% include pictures.html img="0*5TpHrNqaemC7yh-U" alt="" %}*Photo by Fotis Fotopouloson unsplash.com*

***

### Usual Implementation problem

1. Typical **Event subscriptions** where observers (or listeners) register with a subject (or publisher) to receive events have a usual issue in most implementations which is commonly known as the ***Lapsed listener problem***
2. This requires explicit deregistration, as in the typical ***dispose pattern paradigm.***
3. The **leak happens** when an observer fails to unsubscribe from the subject when it no longer needs to listen. Consequently, the subject still holds a strong reference to the observer which prevents it from being garbage collected.

***

### The Usual Resolution Mechanism

1. This can be prevented by the subject holding weak references to the observers, allowing them to be garbage collected as normal without needing to be unregistered.
2. Thus the idea is to keep only the *std::weak\_ptr* in event dispatcher and do the **(lazy) cleanup on event invocation**.
3. Probably something along the lines of -
{% include pictures.html img="1_CT7cfFrVE4Wg5cWPYnbKSQ" alt="Approxmiate event dispatcher: common resolution mechanism" %}*Approxmiate event dispatcher: common resolution mechanism*

4. And a simple notifier would look something like this

{% include pictures.html img="1_mL3H3spS721BbA0B2bBuNw" alt="Sample Notifier for the event dispatcher" %}*Sample Notifier for the event dispatcher*

OR we can opt for the classic “erase and update” using iterator based loop and merge these two loop operation(s).

> There is an issue with this approach, what’s that?
{% include pictures.html img="1_3zb5di6F074tfgGhQ_Q1-w" alt="What seems to be the problem?" %}*What seems to be the problem?*

***

### The Trouble in Paradise

> **What if the observer tries to update you while being notified ???**Suppose something in an observer you fire, ends up in -

1. Adding or removing observers? => Bad (**includes crashes**!)
2. Blocking the action of another thread, who blocks on trying to add an observer? => Bad (**deadlocks**!)
{% include pictures.html img="1_IQoHqec3aVyY9i84tytCYQ" alt="Noooooo" %}*Noooooo* 

Basically the problem is with **Reentrancy** here.

1. Never, ever leave control of your code while holding the lock.
2. Holding a lock and calling a callback is a no-no.

***

### Tying the loose ends (Simple fix)

**“Copy and Broadcast”** — While copying, remove expired observers (from original and hence excluded from the copy), then fire off observers from the copied list.

After this fix, a notifier would roughly look something like -

{% include pictures.html img="1_MKc-QgKtFgeZsAPZv9c0Gg" alt="opy And Broadcast mechanism" %}*Copy And Broadcast mechanism*

***

### Tying the loose ends (Better but complex fix)

1. Maintaining a non-blocking lock-free task queue that includes add/remove/notify etc. events.
2. In case the event handlers take good amount of time, it might be considered launching them “all at once”, i.e. in their own threads over which *join* takes place, or maybe using *executor* mechanism etc.

***

### TL;DR

<iframe src="https://www.youtube.com/embed/eP5zOeDFzl4" width="600" height="375" frameborder="0"></iframe>

***
