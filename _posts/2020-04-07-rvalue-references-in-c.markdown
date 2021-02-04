---
layout:	post
title:	"Rvalue references in C++"
date:	2020-04-07
categories: [ 'C++', 'Interview Preparation' ]
image: 'img/1_e5WJhg2seFQhG4vGd65yeQ.jpeg'
tags: [ featured ]
author: admin
---

  ![](/img/1_e5WJhg2seFQhG4vGd65yeQ.jpeg)*rvalue references*
  
***

### Contents

1. Introduction & Backstory
2. Lvalues and Rvalues
3. Lvalue references & Rvalue references
4. What’s the point of rvalues?
5. How to get an rvalue reference from anything?
6. Usage of rvalues and rvalue references
7. Classic confusion of lvalue and rvalue usage

***  

### Introduction

In quest of performant code, C++11 introduced move semantics and to achieve that, a new terminology & tool — **rvalue references.**

If you find yourself always confused about lvalues, rvalues, and their references, this is the right article is for you. If you know them well, I hope you’ll be kind enough to let me know your feedback and suggestions.

***  

### Backstory

Prior to C++11, only one type of reference existed in C++, and so it was known as “reference”. For better distinction, the ordinary reference X& is now called an *lvalue reference*.

***  

### lvalues and rvalues

In C++, broadly speaking every **expression** can be categorized as an lvalue or an rvalue —

**lvalue —** The expression that refers to a specific memory location. So these are designated objects (that may be referred by their names) and you are allowed to take its address using ‘&’ operator as all lvalues have assigned memory addresses.


```
  =========================================================

  int y = f(x) // x and y are object names and are lvalues.  
  =========================================================

  vector<int> vec;  
  vec[0] //vec[0] is also an lvalue   
  =========================================================

  int i = 2;  
  int j = ++i // Here ++i is lvalue  
  =========================================================
```


When lvalues were originally defined, they were defined as *“values that are suitable to be on the left-hand side of an assignment expression”*. However once the const keyword was added to the C++, lvalues were split into —   
1) modifiable lvalues  
2) non-modifiable lvalues, which are const.

**rvalue -** The expression that refers to a **disposable** **temporary** object so they can’t be manipulated at the place they are created and are soon to be destroyed. An address can not be taken of rvalues.


> rvalues can be realized as everything that is not an l-value

```
  =========================================================

  Myclass g(UserClass()) // Here UserClass() is an rvalue  
  =========================================================

  int i = 1; // 1 is an rvalue  
  =========================================================

  int j = i--; // Here i-- is an rvalue
  =========================================================
```

***

> An rvalue has no name as its a temporary value.

***  

### Lvalue references

A reference that **binds to an lvalue,** lvalue reference is marked with a single ampersand (&). They act as an alias of the bound object.

![](/img/1_yTSGKUlIEfiH9tZA7Akbsg.webp)*The thing about lvalue references (Table#1 and Table#2*

```
  =========================================================  
   
  int x{};  
  const int y = 0; // or const int x {0};
 
  =========================================================
 
  // l-value references (Table 1)  
  int &ref1{ x }; // OK  
  int &ref2{ y }; // Error  
  int &ref3{ 5 }; // Error
 
  =========================================================  
 
  // lvalue references to const (Table 2) 
  const int &ref4{ x }; // OK  
  const int &ref5{ y }; // OK but can't modify  
  const int &ref6{ 5 }; // OK  
  int &ref7{ 5 }; // Error
 
  =========================================================
```
***  

### Rvalue references

A reference that **binds to an rvalue.** rvalue references are marked with two ampersands (&&). They extend the life of the rvalue temporary objects.


> *Every rvalue ref is denoted by ‘&&’ but vice versa is not true.   
> ‘&&’ may also imply forwarding (a.k.a) universal reference. Will cover Universal references in another article.*![](/img/1_P44tpDonN4ti5N7Bvfaoog.webp)*The thing about rvalue references (Table#3 and Table#4)*

```
  =========================================================

  int x{};  
  const int y = 0; // or const int x {0};

  =========================================================

  // r-value references (Table 3)  
  int &&ref7{ x }; // Error  
  int &&ref8{ y }; // Error  
  int &&ref9{ 5 }; // OK  
  ref9 = 6; // OK
 
  =========================================================

  // r-value references to const (Table 4) 
  const int &&ref10{ x }; // Error  
  const int &&ref11{ y }; // Error  
  const int &&ref12{ 5 }; // OK but can't modify  

  =========================================================

```
***

> Lvalue references can bind to Lvalues and Rvalues.  
> Rvalue references can only bind to Rvalues.

***  

### What’s the point of rvalues?

Rvalue references add the possibility to express a new intention in code: — **disposable objects**. When someone passes it over to you (as a reference), it means **they no longer care about it**.

For instance, consider the rvalue reference that this function takes:
```
  =========================================================
  
  void f(MyClass&& x)  
  {  
    ... 
    // Caller doesn't care about x. do whatever you want with it.  
  }

  =========================================================
```
> Thus rvalues can be useful for —   
> 1. Improving performance  
> 2. Taking ownership of an object

***  

### How to get an rvalue reference from anything?

To move the object that the lvalue reference points to, one can move the object directly. We can cast an lvalue explicitly into an rvalue reference. This is what *std::move* does —


> *std::move* is used to convert to rvalue reference, it is exactly equivalent to a static\_cast to an rvalue reference.

```
  =========================================================

  int x = 1, int y = 1;

  int &&ref1 = x; // Error, rvalue can't bind to lvalue

  int &&ref2 = std::move(x); // This works fine
  int &&ref3 = static_cast<int&&>(y) // Also works fine
  =========================================================
```
***  

### Usage of rvalues and rvalue references

rvalue references are more often used as function parameters. This is most useful for function overloads when you want to have different behavior for lvalue and rvalue arguments.

```cpp
=========================================================

void fun(const int & lref) // lvalue arguments will select this  
{
  std::cout << "l-value reference to const\n";
}

void fun(int && rref) //rvalue arguments will select this function 
{
  std::cout << "r-value reference\n";
}

int main() {
  int x { 5 };
  fun(x); // lvalue argument calls l-value version of function  
  fun(5); // rvalue argument calls r-value version of function  
  return 0;
} 
=========================================================
```

**Rule 1:** Do not write **&&** to return type of a function, and there is no need to return a local variable using std::move. When returning a local variable, it is automatically moved in C++11 onwards. However, if the local variable is static, then it is not moved as expected.

**Rule 2:** To use a temporary value, overload the intended function with rvalue reference and move the content.

**Rule 3:** When you write a class with the copy constructor and the assignment operator, write also the move constructor and the move assignment operator. Feel free to refer to [**this article**](https://medium.com/pranayaggarwal25/move-semantics-269e73287b63) for more details.

**Rule 4:** Always use *const* lvalue reference, unless you intend to change the referenced object.

**Rule 5:** Use *std::move* to cast into rvalue references.

***

### Classic confusion of lvalue and rvalue usage

As we have learned above that an rvalue has no name & type of the named object or anonymous object is independent of what it is. So. —


> A named rvalue is an lvalue as all Named objects are lvalues. Only temporary / anonymous objects are rvalues

```
  void f(vector<int> && x) // x is an rvalue reference
  {  
    // ....  
    // here x is an lvalue since it designates the name of an object  
  }
```

> **There can be rvalue references that are themselves lvalues!**

It is common to misunderstand x as a rvalue reference when using it inside a function body like above. Here inside function bodyx itself is not an rvalue reference object because it has a name and named rvalue is an lvalue.  

> **&&** only implies that it is OK to dispose of *x*.

At the place where you intend to move x, you must use *std::move* manually to move it, otherwise, it gets copied.

> Values return by functions/methods and expression are temporary values, so you do have to use **std::move** to move them **(C++ standard to convert to rvalue)** when you pass them to functions/methods that take an rvalue reference as an argument.

Take a look at the below code for example —

```cpp
=========================================================
void fun(const int &lref) // lvalue arguments will select this 
{  
 std::cout << "l-value reference to const\n";  
}

void fun(int &&rref) // rvalue arguments will select this function
{  
 std::cout << "r-value reference\n";  
}
=========================================================

int main()  
{  
 int x{ 5 };  
 fun(x); // lvalue argument calls lvalue version of function  
   
 int &&ref{ 5 };   
 fun(ref);// calls lvalue version of function!!! we didn't move it
 fun(std::move(ref));// calls r-value version of function return 0;  
}  
=========================================================
```
***  

### References

1. <http://www.legendu.net/en/blog/lvalue-rvalue-reference/>
2. <https://www.learncpp.com/cpp-tutorial/15-2-rvalue-references/>
3. <https://www.fluentcpp.com/2018/02/06/understanding-lvalues-rvalues-and-their-references/>


Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com), Have a great day!

  