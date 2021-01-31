---
layout:	post
title:	"Passing smart pointers shared_ptr and unique_ptr"
date:	2020-06-17
categories: [ 'C++' ]
image: img/0*MKKxnMeiF8CTX0zm
author: admin
---

  General guidelines to follow for passing shared\_ptr and unique\_ptr as function arguments and at the time of return.

![](/img/0*MKKxnMeiF8CTX0zm)
> As always, prefer unique\_ptr to shared\_ptr, unless you intend to share ownership.

***

### Introduction

1. Always ask yourself “Do I really need to pass a smart pointer ?”
2. std::unique\_ptr usage
3. std::shared\_ptr usage
4. How to pass correctly pass raw pointer/reference?
5. For Returning smart pointers, what is the norm?

***

### Always ask yourself “Do I really need to pass a smart pointer ?”

You should always assess if passing smart pointers as a function parameter is what you really need. In most cases, you just need to use it and be done with it. In those cases, it’s always good to get a raw pointer and pass it on.

Now before we go further, I must tell you that **raw pointers aren’t always bad**. They can be quite useful in many cases as we’ll read in this post.

***

#### **Guideline**


> Don’t pass a smart pointer as a function parameter unless you want to use or manipulate it, such as to share or transfer ownership.
> **Prefer **passing objects by non-owning raw pointers (*) OR references (&).

#### Reason and Example

```cpp
void f( widget* );   
void f( widget& ); 
```

They stay agnostic of whatever lifetime policy the caller use and are good to observe an object whose lifetime we know exceeds that of the pointer or reference.

This is restrictive and can’t beeasy to commit always then we’ll see and asset the best possible way to pass smart pointers.

![](/img/1*CiD2BpQ9eDI1PX9OYMMLAQ.png)*Step1: Know your lifetimes…The full picture would keep becoming more clear :)*

So following should be the guideline if we are sure about the lifetime —

![](/img/1*tuRoK1HFOMg2xkZcSGyTIw.png)*Guideline to follow in case of no share or transfer of ownership and valid lifetime*

***

### std::unique_ptr usage

*By value unique_ptr< type >* assumes ownership of a *widget*

**Reason:** This is the preferred way to express a consuming function, also known as a **“sink”**.

Using *unique_ptr* in this way in both documents and code, enforces well the function call’s ownership transfer (*Expresses a “sink” function)*

Such as —
```cpp
void sink(unique_ptr<widget>); // takes ownership of the widget
// whereas
void uses(widget*); // just uses the pointed object
```

And a bad example is below —

```cpp
void thinko(const unique_ptr<widget>&); // usually not what you want!
```

[](/img/1*hsARhJ6S3gqAHfiNkdIlGQ.png)*unique_ptr passing guidelines*

and our guideline picture is now —

![](/img/1*cp9haQg73JXSEN-xfrJB-g.png)*Step2: Know your transfers … More clear picture :)*

#### **Guideline:**


> Express a “sink” function using a by-value *unique_ptr* parameter. Use a non-const *unique_ptr&* parameter only to modify the *unique_ptr*.

### std::shared\_ptr usage

#### Passing shared_ptr by value only when you are sharing the ownership

```cpp
void f( shared_ptr<widget> ); // only when you want to retain  
 // object and share ownership
```

> Sharing ownership comes with a cost, so make sure you really mean and intend to pay that price.In pass by value, the argument is copied (usually unless temporary) on entry to the function, and then destroy it (always) on function exit.

Passing *shared_ptr* by value means —   
1) A new ***shared_ptr*** will be **copy constructed.**  
2) Ref count which is an atomic shared variable gets increased.  
3) ***shared_ptr*** copy gets destroyed at the end of the function.  
4) Ref count which is an atomic shared variable gets decreased.

[ Related post: <https://medium.com/pranayaggarwal25/a-tale-of-two-allocations-f61aa0bf71fc> ]

```cpp
void f( const shared_ptr&<widget> ); // may share ownership  
void f( shared_ptr&<widget> ); // may reset pointerIn the special case where the function *might* share ownership but doesn’t necessarily take a copy of its parameter on a given call, then pass a const-ref to avoid the copy on the calls that don’t need it. 
```
So for ***shared_ptr*** follow this —

![](/img/1*1VklHsi0KmZw17S5o0VW8Q.png) *shared_ptr passing guidelines* And finally, we have our full guideline picture here —

![](/img/1*GZhPV6_u-926_gDzo-_RHg.png) *Step3: The final guideline picture*

### How to pass correctly pass raw pointer/reference?

As we have seen above that functions should prefer to pass raw pointers and references down call chains wherever possible.

At the top of the call tree where you obtain the raw pointer or reference from a smart pointer that keeps the object alive. You need to be sure that the smart pointer cannot inadvertently be reset or reassigned from within the call tree below.

**To do this, sometimes you need to take a local copy of a smart pointer, which firmly keeps the object alive for the duration of the function and the call tree.**

Consider this code, the following should not pass code review:

```cpp

// global (static or heap), or aliased local ...  
shared_ptr<widget> g_p = ...;

void my_code()  
{  
 // BAD: passing pointer or reference obtained from a non-local  
 // smart pointer that could be inadvertently reset somewhere   
 // inside f or its callees  
 f(*g_p); // BAD: same reason, just passing it as a "this" pointer  
 g_p->func();  
}

```

The fix is simple — take a local copy of the pointer to keep a ref count for your call tree:

```cpp
// global (static or heap), or aliased local ...  
shared_ptr<widget> g_p = ...;

void my_code()  
{  
 // **cheap:** 1 increment covers this entire function and call trees   
 auto pin = g_p; // **GOOD:** passing pointer or reference obtained from a local   
 // unaliased smart pointer  
 f(*pin); // **GOOD:** same reason  
 pin->func();  
}

```
***

### For Returning smart pointers, what is the norm?

You should follow the same logic above:


> Return smart pointers if the caller wants to manipulate the smart pointer itself, return raw pointers/references if the caller just needs a handle to the underlying object.If you really need to return smart pointers from a function, take it easy and always return *by value*. That is:

```cpp
std::unique_ptr<Object> getUnique();  
std::shared_ptr<Object> getShared();  
std::weak_ptr<Object> getWeak();
```

There are at least three good reasons for this:

1. **Move Semantics** **—** Smart pointers are powered by move semantics: the dynamically-allocated resource they hold is moved around, not wastefully copied.
2. **Return Value Optimization (RVO)** **—**All modern compilers are able to detect that you are returning an object by value, and they apply a sort of return shortcut to avoid useless copies. Starting from C++17, this is guaranteed by the standard. Returning by reference inhibits that shortcut.
3. **Object deletion probability —** Returning *std::shared\_ptr* by reference doesn't properly increment the reference count, which opens up the risk of deleting something at the wrong time, by incurring the risk of having the object deleted (that may be local) when it goes out of scope in another context.
Thanks to RVO. you don’t need to move anything when returning a *std::unique\_ptr* also.

```cpp

std::unique_ptr<Object> getUnique()  
{  
 std::unique_ptr<Object> p = std::make_unique<Object>();  
 return p;   
 // also return std::make_unique<Object>();  
}
```
***

### References

1. <https://herbsutter.com/2013/06/05/gotw-91-solution-smart-pointer-parameters/>
2. <http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#Rr-uniqueptrparam>
3. <https://www.internalpointers.com/post/move-smart-pointers-and-out-functions-modern-c>
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out my [other articles](https://medium.com/pranayaggarwal25) and [website](http://pranayaggarwal.github.io/), Have a great day!

  