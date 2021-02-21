---
layout:	post
title:	"A tale of two allocations"
description: "C++11 introduced shared pointer to safely allocate dynamic memory in the RAII paradigm. We try to demystify the logical model for shared_ptr and usage of make_shared. So let's try to look into it deeper..."
date:	2019-10-12
categories: [ 'C++' ]
image: '1_C4SOhNW1GC5SBLMXBwZv0g'
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

{% include pictures.html img="1_WZWCjCvEPOcKnCGBQgccbA" alt="Since C++17, A default-constructed weak_this goes along enable_shared_from_this" %}
*Strong and weak count*
> Strong and Weak count(s) are typically incremented using an equivalent of *atomic::fetch\_add* with *memory\_order\_relaxed.*
> Decrementing requires stronger ordering to ensure **safe** destruction.

***

### Logical model for shared\_ptr constructor

If a *shared\_ptr* is constructed from an existing pointer that is not *shared\_ptr* the memory for the control structure has to be allocated.

{% include pictures.html img="1_C4SOhNW1GC5SBLMXBwZv0g" alt="Approximate Memory Lyaout" %}
*Approximate Memory Lyaout*

This Control block is destroyed and deallocated when the last weak ref goes away. A *shared\_ptr* construction approach takes two steps

{% include pictures.html img="1_KmTb1wfpSBhhvZaotpXQxw" alt="2 Step memory allocation approach" %}

***

### Logical model for object construction using make\_shared

make\_shared (or allocate\_shared) Allocates the memory for the control structure *and* the object itself in one single mem block.

{% include pictures.html img="1_IO7opntY7n6XkNZ5vhmKHA" alt="Approximate Memory Layout" %}
The object is then constructed by perfectly forwarding the arguments to its constructor.

{% include pictures.html img="1_BOSY6qzJ5SpSNsIcEnUXzw" alt="Single step memory allocation apporach" %}


***

### Pros make\_shared over shared\_ptr

**Performance:** Reduced number of separate allocations

**Cache locality**: Actions that work with both the count structure and the object itself, will have only half the number of cache misses. (In case cache misses are big issue, we might want to avoid working with single object pointers altogether)

**Order of execution and exception safety:** (concern pre-C++17)

{% include pictures.html img="1_zyLgliQi9k8oCxZApg2eKQ" alt="Sample Code" %}
*Sample Code*

A possible execution ordering is

> 1) new Lhs(“foo”))  
> 2) new Rhs(“bar”))  
> 3) `std::shared_ptr<Lhs>`  
> 4) `std::shared_ptr<Rhs>`

And one important advantage, especially in cases of preC++17 codes, is the execution safety. So look at this snippet. Foo has this function signature, you make the call like this..now before c++17 there is no restriction of argument resolution so one of the possible resolution may look like this — now what is second step throws..you have a leak, right?

**Fix 1:** Use *make\_shared*

{% include pictures.html img="1_dDAiLpInQvLA3MB_rHRjiA" alt="Use make\_shared" %}*Use make\_shared* 


Fix 2: Code expansion

{% include pictures.html img="1_vbmQAIgzrtDici2LNzp8nA" alt="Code expansion" %}*Code expansion*


***

### Pro shared\_ptr vs. make\_shared

**Access to the constructor -** *make\_shared* needs access to the constructor it has to call

**Lifetime of the object storage (not the object itself) —** The second advantage is about the lifetime of the object storage (not the object) this is about the destruction vs deallocation, when the last weak count goes off, then only deallocation would take place. In case of `make_shared` the single block becomes the bottleneck. For large size objects in association with some long life `weak_ptr`, this may become problematic.

{% include pictures.html img="1_ydFxhg1tSs1MK6fVtB_BMw" alt="Approximate Memory Layout With shared_ptr" %}*Approximate Memory Layout With `shared_ptr`*, 
You can also specify a custom deleter, if needed!



**Conclusion : Unless good reason, follow this**

> *As a guideline usually, make\_shared is more preferable to shared\_ptr, but there are cases as stated above where one might need shared\_ptr as well.
{% include pictures.html img="1_7FjUG_9-Te4zOXxrX9jX7A" alt="Choose well we must" %}
*Choose well we must*

***

### References
1. [*https://arne-mertz.de*](https://arne-mertz.de/)

  