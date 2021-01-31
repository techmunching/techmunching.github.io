---
layout:	post
title:	"A tale of two allocations"
date:	2019-10-12
categories: [ 'C++' ]
image: 'img/1*C4SOhNW1GC5SBLMXBwZv0g.png'
tags: [ featured ]
author: admin
---

  make\_shared vs shared\_ptr

<iframe src="https://www.youtube.com/embed/h-U9L67R1jE" width="600" height="375" frameborder="0"></iframe>

***

### Quick Recap:

Just a quick recap, shared pointers work on the concept of ref count, they maintain a separate control block that stores these count.

The way shared\_ptr works is that they maintain -

**strong reference count (S)** — number of shared\_ptr(s) keeping the object alive. The shared object is destroyed (and possibly deallocated) when the last strong ref goes away.

**weak reference count (W)** — number of active weak\_ptr(s) currently observing the object + (S!=0)

![Since C++17, A default-constructed weak_this goes along enable_shared_from_this](/img/1*WZWCjCvEPOcKnCGBQgccbA.png)Strong and weak count
> Strong and Weak count(s) are typically incremented using an equivalent of *atomic::fetch\_add* with *memory\_order\_relaxed.*
> Decrementing requires stronger ordering to ensure safe destruction.### Logical model for shared\_ptr constructor

If a shared\_ptr is constructed from an existing pointer that is not shared\_ptr the memory for the control structure has to be allocated.

![](/img/1*C4SOhNW1GC5SBLMXBwZv0g.png)Approximate Memory LyaoutThis Control block is destroyed and deallocated when the last weak ref goes away. A shared\_ptr construction approach takes two steps

![](/img/1*KmTb1wfpSBhhvZaotpXQxw.png)2 Step memory allocation approach### Logical model for object construction using make\_shared

make\_shared (or allocate\_shared) Allocates the memory for the control structure *and* the object itself in one single mem block.

![Approximate Memory Layout](/img/1*IO7opntY7n6XkNZ5vhmKHA.png)Approximate Memory LyaoutThe object is then constructed by perfectly forwarding the arguments to its constructor.

![Single step memory allocation apporach](/img/1*BOSY6qzJ5SpSNsIcEnUXzw.png)### **Pros make\_shared over shared\_ptr**

**Performance: **Reduced number of separate allocations

**Cache locality**: Actions that work with both the count structure and the object itself, will have only half the number of cache misses. (In case cache misses are big issue, we might want to avoid working with single object pointers altogether)

**Order of execution and exception safety:** (concern pre-C++17)

![](/img/1*zyLgliQi9k8oCxZApg2eKQ.png)Sample CodeA possible execution ordering is


> 1) new Lhs(“foo”))  
> 2) new Rhs(“bar”))  
> 3) std::shared\_ptr<Lhs>  
> 4) std::shared\_ptr<Rhs>And one important advantage, especially in cases of preC++17 codes, is the execution safety. So look at this snippet. Foo has this function signature, you make the call like this..now before c++17 there is no restriction of argument resolution so one of the possible resolution may look like this — now what is second step throws..you have a leak, right?

**Fix 1: **Use make\_shared

![](/img/1*dDAiLpInQvLA3MB_rHRjiA.png)Use make\_sharedFix 2: Code expansion

![](/img/1*vbmQAIgzrtDici2LNzp8nA.png)Code expansion### **Pro shared\_ptr vs. make\_shared**

**Access to the constructor - **make\_shared needs access to the constructor it has to call

**Lifetime of the object storage (not the object itself) — **The second advantage is about the lifetime of the object storage (not the object) this is about the destruction vs deallocation, when the last weak count goes off, then only deallocation would take place. In case of make\_shared the single block becomes the bottleneck. For large size objects in association with some long life weak\_ptr, this may become problematic.

![](/img/1*ydFxhg1tSs1MK6fVtB_BMw.png)Approximate Memory LayoutWith shared\_ptr, You can also specify a custom deleter, if needed!

**Conclusion : Unless good reason, follow this**


> *As a guideline usually, make\_shared is more preferable to shared\_ptr, but there are cases as stated above where one might need shared\_ptr as well.*![](/img/1*7FjUG_9-Te4zOXxrX9jX7A.png)References

[**Meme Generator**  
*The Fastest Meme Generator on the Planet. Easily add text to images or memes. It's a free online image maker that…*imgflip.com](https://imgflip.com/memegenerator "https://imgflip.com/memegenerator")[](https://imgflip.com/memegenerator)[*https://arne-mertz.de*](https://arne-mertz.de/)

  