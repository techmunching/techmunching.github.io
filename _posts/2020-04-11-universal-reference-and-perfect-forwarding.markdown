---
layout:	post
title:	"Universal Reference and Perfect forwarding"
date:	2020-04-11
categories: [ 'C++' ]
image: img/0*wi8CKG57G8q3-woE.gif
author: admin
---

  **Also known as Forwarding reference**

![](/img/0*wi8CKG57G8q3-woE.gif)

***
**Prerequisites:** [Rvalue references](https://medium.com/pranayaggarwal25/rvalue-references-e99dfd3933ff)

#### Table of Contents:

1. Introduction  
2. The double life of “&&”  
3. How to identify a Universal Reference  
4. Universal Reference in templates  
5. Universal Reference in auto  
6. How does the magic of Univeral Reference work?  
7. Handle with care  
8. What is the need? The perfect forwarding problem  
9. How to work with Universal References?  
10. Universal Reference in “typedef”  
11. Universal Reference in “decltype”  
12. The ultimate Truth

***

### Introduction


> Every rvalue ref is denoted by *&&* but vice versa is not true.

Rvalue reference ⇒ “*&&*”   
“*&&*” **⇏** Rvalue reference.

“*&&*” in source code though may have the syntactic *appearance* of an rvalue reference (“*&&*”), but the meaning of an lvalue reference (“&”).

***

### The double Life of “&&”

There is a double life that “&&” lives on and we need to understand that. Let’s see some examples —

![](/img/1*BejVgArr4hHBus4hS3aoSw.png)*The double life of “&&”*

In “*type &&*”, *&&* means either -

**1. Rvalue reference —** As you’d expect. 
i) Binds rvalues only  
ii) Facilitates the moving of objects.

**2. Forwarding reference** (a.k.a **Universal reference**) — The mentioned type is a special type of reference that can bind anything & everything.

i) Universal reference can mean both Rvalue reference and Lvalue reference.  
ii) May facilitate copying, may facilitate moving.  
iii) Syntactically “type&&” but semantically “type&&” or “type&”.

***

> Universal reference can bind lvalues, rvalues (be it const or non-const) thus everything.

***

### How to identify a Universal Reference


> *If **T** is getting deduced and variable / parameter type is unqualified **T&&**, then only it’s a Universal Reference. Here if **T** has already been deduced, that means no Universal Reference, plain simple rvalue reference.*
> *Not all `T&&` are Universal Reference.*

There are 4 possible contexts —   
1. Function template parameters —

```cpp

template<typename MyType>  
void f(MyType&& param); // && ≡ rvalue reference

```
2. **auto** declarations *auto&& var2 = var1;*  
3. typedef declarations  
4. decltype expressions

Out of the above 4, the first two are the most commonly occurring cases. As we have seen in the rule, type deduction is must so —

![](/img/1*hxu81px55zDERu49GRh3bA.png)*Understanding Type Deduction*

***

### Univeral references in Templates

```cpp
template<typename T>  
void f(T&& param)  
{  
 std::cout<< __PRETTY_FUNCTION__ << std::endl;  
}
```
So calling function ‘f’ in different ways will produce different results.

**Example 1:** Calling function ‘f’

```cpp

Widget w;  
f(w);   

// T&& forwarding reference will be resolved as Lvalue Ref  
// Thus instantiating void f(Widget&) out of the template 

void f(T&&) [with T = Widget&]. // Printed

```

**Example 2:** adding const also —

```cpp

const Widget cw;  
f(cw);   

// T&& forwarding reference will be resolved as Lvalue Ref to const  
// Thus instantiating void f(const Widget&) out of the template 

void f(T&&) [with T = const Widget&]. // Printed

```
**Example 3:** passing rvalues —

```cpp

f(Widget());  

// T&& forwarding reference will be resolved as Rvalue Ref  
// Thus instantiating void f(Widget&&) out of the template

void f(T&&)[with T = Widget] // Printed, Why?? explained later in Table 2

```

**Example 4:** adding const with rvalues

```cpp

const Widget cw;  
f(std::move(cw));  

// T&& forwarding reference will be resolved as Rvalue Ref to const  
// Thus instantiating void f(const Widget&&) out of the template

void f(T&&)[with T = const Widget]   
// Printed, Why? Explained later in Table 2

```
***

### Univeral references in auto
```cpp
*auto&& var = 10; // 10 is value hence var being 
// a Universal reference resolves to an rvalue reference.
```
Type of var is *int &&*

```cpp
std::vector<int> vec;  
auto&& var = vec[0]; // vector[] is int& hence var being 
// a Universal reference resolves to an Lvalue reference.
```
 Type of var is *int&*.

```cpp
int i = 10;  
const auto&& var = i; // Not a Universal Reference hence Error!!
```
***

### How does the magic of Univeral Reference work?

In pre-11 C++, it was not allowed to take a reference to a reference: something like A& & would cause a compile error. C++11, by contrast, introduces the following **R*eference collapsing rules***:

![](/img/1*kMsWQ7At7DM5eNqZt9QcQg.png)*Table 1: Reference Collapsing Rules*

Reference collapsing is magic that enables Universal references to work. These rules are basically a logical AND of Lvalue ref implying ‘0’ (zero) with Rvalue ref implying ‘1’(one)


> *Reference Collapsing Rule#1: Lvalues are infectious.*

So basically in the above template examples of Univeral reference —

![](/img/1*QNPrhEdAboP8AubgIxuhdg.png)*Table 2: Reference Collapsing Rules for template Examples 1- 4*

Columns with color code orange here don’t have && for T. The reason behind that is —


*“In template / auto type deduction, references become non-references before lvalue / rvalue analysis. Once deduction takes place, analysis occurs and if lvalue, reference is put back on”*

Also, looking at the table values leads to another rule which is —


> *Reference Collapsing Rule#2: const volatile qualifiers of original type are reserved.*

**Remember, It’s still impossible to declare a reference to reference explicitly. The compiler can do it, you can’t!!**

***

### Handle with care…

- The thing with overloading used with universal reference is that it almost always turns out differently than expected.

```cpp
=========================================================  

class MyWidget {  
 ...template<typename T> // **Version 1**  
 void f(const T& rhs); // Best match for const Lvalue Reftemplate<typename T> // **Version 2 **  
 void f(T&& rhs); //** universal reference, can handle everything**  
   
 ...  
};

=========================================================

MyWidget m;  
const MyWidget cm;

f(m); // Case 1, Calls Version 2!!

f(std::move(m)); // Case 2, Calls Version 2  

f(cm); // Case 3 Calls Version 1  

f(std::move(m)); // Case 4, Calls Version 2

=========================================================
```
This happens as Version 2 of ‘f’ with Universal reference can resolve to anything if needed and thus is a better match for Lvalue Reference. However, if an exact match is found like in case 3, there Version 1 of ‘f’ is called.

- Another precaution with Universal Reference is that you can’t forward NULL as null pointers because that will be treated as int 0 = (zero). Another reason to use std::nullptr !!

- If braced initializers are passed, the compiler can not deduce the type so you should explicitly mention the type.

***

### What is the need? The perfect forwarding problem

Let *func(E1 E2, ... En);* be an arbitrary function call with generic parameters E1, E2, …, En. We’d like to write a function wrapper such that *wrapper(E1 E2, ... En)* is equivalent to *func(E1 E2, ... En)*.

The first approach that comes to mind is —

```cpp
template <typename T1, typename T2>  
void wrapper(T1 e1, T2 e2) {  
 func(e1, e2);  
}
```

But this will not work if ‘func’ accepts its param by reference since ‘wrapper’ is passing its local copies. We can add another definition.

```cpp
template <typename T1, typename T2>  
void wrapper(T1& e1, T2& e2) {  
 func(e1, e2);  
}
```
but what if we would want to pass rvalues also? What about const-ness?  
As you see this will be an exponential problem. Brute force solution —

![](/img/1*cnR_CDZLMixWJChrha8nLg.png)*These are just lvalue versions, rvalue versions will also be needed*

Hence there was a need for perfectly forwarding whatever parameter is received and that is achievable by Universal references.

***

### How to work with Universal References?


> A type in independent of lvalueness and rvalueness

There are lvalues of type int (e.g., variables declared to be ints), and there are rvalues of type int (e.g., literals like 10)
```cpp

Widget&& var1 = makeWidget() // var1 is an lvalue, but  
// its type is rvalue reference (to Widget)

```
As with Rvalue references, we use *std::move*, with Universal references we should always use [`std::forward<T>`](https://en.cppreference.com/w/cpp/utility/forward) present in <utility> header which takes care of copy in case of Lvalue Ref and move in case of Rvalue Ref.

`std::forward` keeps the reference type of x. So:

* If `x` is an rvalue reference then `std::forward` is = `std::move`,
* If `x` is an lvalue reference then `std::forward` doesn’t do anything.
So for the previous example of `wrapper` function on *func(E1 E2, ... En)* -

```cpp

template <typename T1, typename T2>  
void wrapper(T1&& e1, T2&& e2) {  
  func(std::forward<T1>(e1), std::forward<T2>(e2));  
} // Awesome!!

```
***

> std::forward is a conditional cast but std::move is an unconditional cast.Applications of perfect forwarding are `std::make_shared` , `std::make_unique` , `vector::emplace_back` etc.

***

### Universal references in “typedef”
```cpp
template<typename T>  
class Widget  
{  
// ...  
}

=========================================================

Widget<int&> w;

typedef Widget&& UniRefToWodget; 

UniRefToWodget &v1 = w;   
// Reference collapsing, v1's type is Widget&

const UniRefToWodget &v2 = std::move(w);   
// Reference collapsing, v2's type is const Widget&

UniRefToWodget &&v3 = std::move(w);   
// Reference collapsing, v3's type is Widget&&

```

***

### Universal References in “decltype”

For *decltype,* Type deduction rule is different.

1. decltype(id) ⇒ id’s declared type.
2. decltype(non-id lvalue expression) ⇒ Expression’s type, **Lvalue Ref (T&)**
3. decltype(non-id rvalue expression) ⇒ Expression’s type, **Non-Ref (T)**
![](/img/1*wBdb9YCnAO0VPvwqLWnbJA.png)*decltype and Universal Reference*

### Ultimate Truth

“*&&*” is really Always Rvalue Reference, but due to the magic of Reference collapsing, It can work as a reference that can work with anything hence the fancy name “Universal Reference” a.k.a. “Forwarding Reference”. :)

### Bonus point : coz you made it till here!!

What does `std::forward` do actually?

![](/img/1*hLPj-EPcjNl2afshgBLaWw.png)*Implementation of std::forward*

**Case 1:** passing lvalues sayint& , T is deduced as int&

![](/img/1*p3CTuZ7DnDKASZlChPn3rg.png)*Putting T as int&*

After Reference collapsing, *std::forward* turns into this, which is as good as passing lvalue.

![](/img/1*eMkkMxSVP01Aq-uzj1okmw.png)*The final definition looks like this after Reference collapsing*

**Case 2:** passing rvalues say *int&&* , T is deduced as int (Why? Read note mentioned after Table 2). 

Now After applying T as int —

![](/img/1*rqib7b-jjZtkHYHDf3neZg.png)*The final definition looks like this after Reference collapsing*

*std::forward* could do the job without [std::remove\_reference](https://en.cppreference.com/w/cpp/types/remove_reference). Reference collapsing does the job already, so `std::remove_reference<T>` is superfluous.

But it’s there to turn the T&t into a non-deducing context, thus forcing us to explicitly specify the template parameter when calling *std::forward* .

### References

1. <https://isocpp.org/blog/2012/11/universal-references-in-c11-scott-meyers#NittyGrittyDetails>
2. <https://channel9.msdn.com/Shows/Going+Deep/Cpp-and-Beyond-2012-Scott-Meyers-Universal-References-in-Cpp11>
3. [http://thbecker.net/articles/rvalue\_references/section\_08.html](http://thbecker.net/articles/rvalue_references/section_08.html)
4. <https://www.fluentcpp.com/2018/02/06/understanding-lvalues-rvalues-and-their-references/>
5. <https://eli.thegreenplace.net/2014/perfect-forwarding-and-universal-references-in-c>
6. <https://www.slideshare.net/oliora/hot-universal-references-and-perfect-forwarding-82155460>
7. [http://thbecker.net/articles/rvalue\_references/section\_01.html](http://thbecker.net/articles/rvalue_references/section_01.html)
  