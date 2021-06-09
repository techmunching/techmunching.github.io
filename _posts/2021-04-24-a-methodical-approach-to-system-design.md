---
layout: post
title: "Approaching System Design for Interviews Methodically"
description: "Guide to learn system design Methodically for interviews - The only system design 101 that you will need to go through for your interviews
The premium free course and template to answer any system design question — A definitive method to tackle any system design problem such as Design Youtube, URL shorterner, Instagram in a unique way. Learn and apply CAP theorem, scalability, latency, throughput as easy as it can be."
date: 2021-04-24
categories: ["System Design", "Programming", "Interviews"]
image: "photo-1614439007888-c8f7ae337eb1"
tags: [featured]
author: admin
---

# Dreaded night before the interview

A few weeks ago, my friend got a chance to interview at _Google_.

Nitin is an accredited software developer from IIT working at a big product based software development firm (such as FAANG). For me, he belongs firmly in the category of 'Big Deals' in the programming sphere. So before the interview day, he calls me up:

{% include pictures.html img="photo-1598979072634-26e2d8821e68" alt="Interview at FAANG" %}

"I wish system design was as easy as recurssion, wherein you have a base-case following which you perform the core computation and conclude with a result, you get it right? Something that I could estimate, I knew the next step for?"

Relatable, right? It isn't that System Design is difficult, however, it's just the case that either you haven't had a chance to work on large scale architectures or else you worked on just a tiny part of it. In both the scenarios, we end up being not so confident about our readiness for a system design interview rounds.

Take for example, I ask you to represent 32 as a power of 2. You could immediately go on a step by step method of doing that, few possible solutions being:
<img src="https://latex.codecogs.com/svg.image?\log_2&space;32\:=\:?,\:answer\:=\:5" title="\log_2 32\:=\:5,\:answer\:=\:5" width="40%" />
<img src="https://latex.codecogs.com/svg.image?32\:/\:2^?\:=\:1,\:answer\:=\:5" title="32\:/\:2^?\:=\:1,\:answer\:=\:5" width="40%" />

In both the scenarios, you exactly knew what the next steps are and you could execute on those.

---

## We need a design system for system design

Today, with enornous number of articles/books/lectures on system design and yet we are lacking a methodical approach, a step-by-step journey to master every system design question and have an appropriate discussion. A user manual for System Design could end up being so helpful to countless software developers, making system design interviews a breeze.

{% include pictures.html img="vector-21731841" alt="System Design Masterclass and guide" %}

---

## Introducing, TechMunching - Hitchhiker's guide to System Design

<br>
To read Hitchhiker's guide is to step into the alien world of Systen Design with yoda guiding your soul through a perplexing world, assuring you "Don't panic".
</br>
At TechMunching, henceforth, our mission is to instill confidence allowing you to lead and be in-charge of every system design discussion. In Hitchhiker's guide to System Design, we explain the concepts with both a _explain me like I am 5_ and the _Rocket Science_ version.

> Innovation is taking two things that already exist and putting them together in a new way.

This is the first of a multi-part series of the complete guide. Here is a little glimpse of what you see in this journey:

1. Applying a generic template to guage the problem
2. Acronyms at each step, so that you don't miss anything
3. Availbility of cheat-sheet for calculations, MAU, DAU, DAWU
4. Breaking each problem into known categories like big image (Instagram), small video (Tik Tok, Reels), so that concepts can be inter-related
5. Yardstick to measure the success and failure of the solution you are presenting
6. Code for common patterns cache, load balancer, hash functions, authentication, cookie, redirection
7. Questions to ask at each step which doesn't overwhelm you or the interviewer
8. Higher package and a Happy life :P

We value every comment provided to us. Hence, write down in comments below if your have doubts or suggestion.

---

## Applying the knowledge

Now let's try to embark on this journey, with the a simple and a hard problem. I am sure we all have seen and read these problem multiple times. But what differs here is the approach and steps we take to make it easy to execute and remember.

In the capacity of current post, we would be focussing on requirements and capacity estimation.

### URL Shortening

Referring to the template section 1., we remind ourselfs to have 3 types of requirements (remember acronym ENF - enough)

#### Requirements [Template: [Section 1](#section-1-requirements-quick-look-sheet)]

##### Functional Requirements for URL shortener

- Generate a short and unique alias of any URL.
- When users hit the short link, our system should redirect them to the original link.
- Links may expire after a duration and users can specify the expiration time.
- Custom link may or may not be included in this set of requirements.

##### Non-functional Requirements for URL shortener

- The system should be highly available (In case, the service is down, all redirections will fail).
- URL redirection should happen in real-time (It should be lightning fast).
- Shortened URLs should not be easily predictable (from a security point of view).

##### Extended Requirements for URL shortener

- Building REST system for B2B offering
- Logging and analytics

#### Capacity Estimation [Template: [Section 2](#section-2-capacity-quick-look-sheet)]

##### Behaviour estimation

As shortened URL would be used by many users once shortened, our system would be a read-heavy system.
We define that our system is text content based and small content size in [table](#img-in-content).
Let’s assume, one user may request for a new URL and use it 100 times for redirection.
So, the ratio between write and read would be:
<img src="https://latex.codecogs.com/svg.image?write\::\:read\:=\:1\::100" title="write\::\:read\:=\:1\::100" />

##### Traffic estimation

How many URL requests is the service capable of handling?

Reading from [table](#img-in-dau), we see that our daily active users are `100K`, which concludes are monthly requests to:
<img src="https://latex.codecogs.com/svg.image?dau\:*url\:writes\:per\:user\:*\:30\:days\:=\:100K\:*\:100\:*\:30\:=\:300M" title="dau\:*url\:writes\:per\:user\:*\:30\:days\:=\:100k\:*\:100\:*\:30\:=\:300million" />

Further, calculating the URL read redirections from the short link:
<img src="https://latex.codecogs.com/svg.image?total\:monthly\:writes\:*\:read\:ratio\:=\:300\:M\:*\:100\:=\:30\:B\:url\:per\:month" title="total\:monthly\:writes\:*\:read\:ratio\:=\:300\:million\:*\:100\:=\:30\:billion\:url\:per\:month" />

For URL per second calculation. You could also use 2.5M as the estimation for 30 \* 24 \* 3600 as per [table](#img-for-latency):

<img src="https://latex.codecogs.com/svg.image?300\:M\:/\:(30\:days\:*\:24\:hours\:*\:3600\:secs)\:=\:120\:url\:/\:sec" title="500\:million\:/\:(30\:days\:*\:24\:hours\:*\:3600\:secs)\:=\:200\:url\:/\:sec" />

URL redirections:

<img src="https://latex.codecogs.com/svg.image?url\:redirection\:*\:read\:ratio\:=\:120\:*\:100\:=\:12K\:url\:redirections\:/\:sec" title="200\:*\:100\:=\:20k\:url\:redirections\:/\:sec" />

##### Bandwidth estimation:

Let's say that each stored object is approximately 500 bytes, please refer [table](#img-for-content) for grasping the estimation.

For write requests:
<img src="https://latex.codecogs.com/svg.image?120\:*\:500\:bytes\:=\:60KB\:/\:sec" title="120\:*\:500\:bytes\:=\:60KB\:/\:sec" />

For read requests:
<img src="https://latex.codecogs.com/svg.image?12K\:*\:500\:bytes\:=\:6MB\:/\:sec" title="12K\:*\:500\:bytes\:=\:6MB\:/\:sec" />

##### Data storage estimation:

Let’s assume, the system stores all the URL shortening requests and their shortened link for 5 years. As we expect to have 300M new URLs every month, the total number of objects we expect to store will be:
<img src="https://latex.codecogs.com/svg.image?300\:M\:*\:(5\:*\:12)\:months\:=\:18\:billion\:objects" title="500\:M\:*\:(5\:*\:12)\:months\:=\:30\:billion\:objects" />

For that we will need total storage of:
<img src="https://latex.codecogs.com/svg.image?18\:billion\:*\:500\:bytes\:=\:9\:TB" title="30\:billion\:*\:500\:bytes\:=\:15\:TB" />

##### Cache memory estimation:

If we follow the 80–20 rule, meaning we aim to serve 20% request from the cache itself.
Since we have 12K redirections per second:

<img src="https://latex.codecogs.com/svg.image?12K\:*\:3600\:seconds\:*\:24\:hours\:=\:~1\:billion\:requests\:per\:day" title="20K\:*\:3600\:seconds\:*\:24\:hours\:=\:~1.7\:billion\:requests\:per\:day" />

If we plan to cache 20% of these requests, we will need:
<img src="https://latex.codecogs.com/svg.image?0.2\:*\:1\:billion\:*\:500\:bytes\:=\:~100GB\:of\:memory" title="0.2\:*\:1.7\:billion\:*\:500\:bytes\:=\:~170GB\:of\:memory" />

(Actual requests might be duplicated so this number is more than sufficient)

This somes up our requirements and capacity estimations for the URL shortner system design.

### Youtube

Taking step two, we now look how templates makes our calculation generically application to different problems. Let's say we are trying to design Youtube, so our service will be of content type `Video - Lager` which gives us the average size of 100MB as per [table](#img-for-content).

- Content Type: Video - Large
- Average Size: 100MB
- Daily Active writes per user: 0.001
- Daily Active Users: 800M

So our database size for the above example becomes:

{% include pictures.html img="ExampleYoutube" alt="Youtube System Design" %}

---

### Template

<!-- https://www.tablesgenerator.com/markdown_tables -->

<a name="section_1"></a>

##### Section 1. Requirements, Quick look sheet:

{% include pictures.html img="requirementTable" alt="Requirement Table of System Design" %}

##### Functional requirements:

You may call this the system's main goal. A functional statement is described here as a set of outputs and inputs. Let’s say we are able to identify system perform 4 functions, those 4 requirements we should be able to jot down in this step.

Hence what our system functionally is expected to do from users point of view is functional requirements. Once you understand what solution system offers to its users, it is easy to detail out these.

##### Non-functional requirements:

What qualities we want in our system if we wish the business to succeed. As a part of these, we should consider the requirements that need to be examined carefully. Failure to meet this requirement would endanger the project’s business plan. Different system qualities are necessary to fulfill each such non-functional requirement.

While designing our system, we would be keeping these in mind and at the end we’ll assess how good our system is by measuring up against them.

Most of the time one has to make a trade off depending upon what attributes are needed in the system - Performance, Resilience, Availability, Consistency, Scalability, Reliability etc.

##### Extended Requirements:

These might come from the interviewer as we build up our solution, as an extension of the problem or we can also add these if it will be a nice-to-have feature.

##### Section 2. Capacity, Quick look sheet:

{% include pictures.html img="capacityTable" alt="Capacity requirment of System Design" %}

Getting to know the system's scale is an important point of its design. Just like you consider the maximum size of input list of data in a programming problem, for system design you also take input requests and your system load handling into consideration.

If the measurements of the system are very large in number, then they are high scale systems.
So we will try to assess our system based on the following -

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/BBT-DC.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*That's the right acronym for capacity needs - BBT-DC (Big Bang Theory - Dr. Cooper)*

##### Behaviour estimation

We should try to understand whether our system will be read-heavy or write-heavy. What kind of a ratio is going to be there between read / writes?
This will help us in choosing the right distribution, cache and database for our system.

We have categorized various systems based on their content type and size of service (or you many call questions) in limited set of buckets, which then makes it easy to cross apply the concepts from each other.

Content Type based categorization of System Design (System Design cheat sheet):

<h6 style="display=none;" id="img-for-content"></h6>
{% include pictures.html img="ContentType" alt="Content Type based categorization of System Design" %}

##### Traffic estimation

This means how many read and write requests we are going to get per second / per month. We need to devise numbers of the same in order to get the storage and memory requirements later on.

You can refer to the below sheet for quickly estimating the MAU/DAU for your service.
Chart for DAU/MAU (System Design cheat sheet):

<h6 style="display=none;" id="img-for-dau"></h6>
{% include pictures.html img="DAU" alt="Quick look chart for Daily Active Users" %}

##### Data storage estimation

We would also need to identify how much data we would need to store for a duration of in case our business runs for X years. For that we would use an average of per month required capacities once gathered.

<h6 style="display=none;" id="img-for-dawu"></h6>
{% include pictures.html img="DAWU" alt="Number of writes per user per second" %}

##### Bandwidth estimation

Once we have the Inbound / Outbound requests number in place, we need to take an average on how much bandwidth we would need based on the average request size vs average response size.

##### Cache memory estimation

In case, we would like to cache some of the responses to be served be it on-demand OR pre-emptively what is the memory size, we expect to be used for a cache?
We would likely be using 80-20 rule for estimation of caching for this.

###### What Is the 80-20 Rule?

The 80-20 rule, also known as the Pareto Principle, asserts that 80% of outcomes (or outputs) result from 20% of all causes (or inputs) for any given event. In business, a goal of the 80-20 rule is to identify inputs that are potentially the most productive and make them the priority.

### Sneak peak to next article

Till now you have already witnessed a new way to look at system design problems. We have further lectures and stories to share, the below chart allows for figuring out the latency in the system super quickly.

<h6 style="display=none;" id="img-for-latency"></h6>
{% include pictures.html img="Latency" alt="" %}

You could also [sign-up](https://vg562vdcmzh.typeform.com/to/Yf5RxIbw) for limited free seats in our premium system design class taught by experienced software architects.

STAY TUNED.
