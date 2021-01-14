---
layout:	post
title:	"Object construction order in case of virtual and/or multiple inheritance"
date:	2020-07-03
categories: [ 'C++', 'OOPS' ]
image: img/0*VtNlssb8zTqF0nY9.gif
author: admin
---

  How virtual vs. multiple inheritance affect class object construction order?

![](/img/0*VtNlssb8zTqF0nY9.gif)I’ll try to keep this post short and simple 😊

### Table of Contents

1. An example class having virtual and/or multiple inheritance
2. What are the rules for object construction order?
3. Understanding the object construction order rules
4. Let’s apply the rules, shall we?
5. Bonus Question
### Example: A class having virtual and/or multiple inheritance

class Xis declared here as inherited by D1, D2 which in turn are inherited by other class in a complex manner by other classes.

![](/img/1*9ZFGEBfTvwaafG1tsZR4QQ.png)This will be our complex exampleHere as we clearly see following can be established —   
1) V1 inherited by B1  
2) D1 inherited by V1  
3) V2 inherited by B1 and B2  
4) D2 inherited by B3 and V2  
5) X inherited by D1 and D2

### What are the rules for object construction order?

The following set of rules is applied recursively:-

#### 1. First comes virtual

First, the most derived class’s constructor calls the constructors of the virtual base class subobjects. The very first constructors to be executed are the virtual base classes anywhere in the hierarchy.   
Virtual base classes are initialized** In depth-first, left-to-right order.**

#### 2. Then comes multiple inheritance —

After all virtual base class constructors are finished, the construction order is generally from base class to derived class. Direct base class subobjects are constructed **in the order they are declared in the class definition**.

#### 3. Next comes class members

Next, (nonstatic) member subobjects are constructed, in the order, they were declared in the class definition.

#### 4. Finally the constructor body

In the last, the body of the constructor is executed.

Also, one point to be noted is that —


> Whether the inheritance is public, protected, or private doesn’t affect initialization order### Understanding the object construction order rules

The rules are easiest to understand if you imagine that the very first thing the compiler does in the derived class’s constructor is to make a hidden call to the constructors of its virtual base and then non-virtual base classes (hint: that’s the way many compilers actually do it).


> It’s like a DFS where the order on a same level is driven by class definitionExample — class ***D*** inherits from both ***B1*** and ***B2***,

1. The constructor for ***B1*** executes first
2. then the constructor for ***B2***,
3. Then the constructor for ***D***.   
This rule is applied recursively
For example, if ***B1*** inherits from ***B1a*** and ***B1b***, and ***B2*** inherits from ***B2a*** and ***B2b***, then the final order is   
1. ***B1a*** => ***B1b*** => ***B1***, and then   
2. ***B2a => B2b =>*** ***B2****   
3. *and ofcourse in the end* ****D***.

Note that the order ***B1*** and then ***B2*** (or ***B1a*** then ***B1b***) is determined by the order that the base classes appear in the declaration of the class, *not* in the order that the initializer appears in the derived class’s initialization list.

[ Refer <https://medium.com/pranayaggarwal25/using-modern-class-members-and-initializations-c11e931c3ba> for more details about class member initializations ]

### Let’s apply the rules, shall we?

![](/img/1*9ZFGEBfTvwaafG1tsZR4QQ.png)Looks easier now, doesn’t it?The initialization order for a X object in Example 2 is as follows, where each constructor call shown represents the execution of the body of that constructor:

![](/img/1*a-qxlGwcVWru5zB3Xo_4Yw.png)So —

![](/img/1*TTTVf62ztLB7o-Z0g3na2A.png)First — Construct the virtual bases V1 and V2 recursively by rulesAfter that, construct the remaining nonvirtual bases:

![](/img/1*TY1_6ucxEUqM8HNQgeMsgQ.png)Second — Construct the remaining non-virtual bases D1 and D2Next, construct the members ***M1*** and ***M2 —***

![](/img/1*cDhZbtSxlmIfZanCtUjkAA.png)Third — Class member constructionand in the last —

![](/img/1*d0jyqK1WZL9p4MfTVvnRlg.png)Final constructionif you’re a fan of graphics, here is how the inheritance hierarchy looks like :

![](/img/1*lNJNK6S_PTgulnoservZoQ.png)Inheritance and object construction hierarchy (v means Virtual)That’s pretty much it!

![](/img/0*RymMjGO7qYGCJPV2)### Bonus Question Time, yay!! 😉

Thanks for reading it till here.

### What is the exact order of destructors in a multiple and/or virtual inheritance situation?

The exact opposite of the same constructor order.

Reminder to make your base class’s destructor virtual, at least in the normal case. Why, I think you already know why? 😊

When someone says delete using a Base pointer that’s pointing at a Derived object, had Base’s destructor not been virtual, Derived’s destructor would not have been called – with likely bad effects, such as resources owned by Derived not being freed.

### References

1. <http://www.gotw.ca/gotw/080.htm>
2. <https://isocpp.org/wiki/faq/multiple-inheritance#mi-vi-ctor-order>
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out my [other articles](https://medium.com/pranayaggarwal25) and [website](http://pranayaggarwal.github.io/), Have a great day!

  