---
layout:	post
title:	"Lambdas and closures in C++"
date:	2020-10-27
categories: [ 'C++' ]
image: img/0*PA8-5rLUUze-e0iK
author: admin
---

  What are closures vs how lambdas are different from closures?

![](/img/0*PA8-5rLUUze-e0iK)They look similar!!
### Table of Contents
1. Just your everyday functions
2. Lambda Expressions
3. What are closures
4. Lambdas vs. Closures
5. Distinction by Examples
6. Close analog to a closure

***
### Just your everyday functions

When most people think of *functions*, they think of **named functions** such as -

```cpp
std::string a_named_function () { 
    return "This string is returned from a named function"; 
}
// A named function
```

These are called by name, of course:

```cpp
foo(); //returns the string above
```

***
### Lambda Expressions

The term “lambda” is short for *lambda expression*, and a lambda is just that 😊:

1. **Lamdba expression** — An *expression *that specifies an anonymous function object.

2. **Lamdba function** — This term is used interchangeably with the term “lambda expression.”

As such, it exists only in a program’s source code.


> A lambda does not exist at runtime.

A lambda expression **specifies an object, not just a function without a name,** capable of capturing variables in scope.

* Lambdas can frequently be passed around as objects.
* A lambda is essentially a function object that is specified inline.
* In addition to its own function parameters, a lambda expression can refer to local variables in the scope of its definition.

***
### What are closures

\* What is C++ specific part here

*A closure is a general concept in programming that originated from functional programming. When we talk about the closures in C++, they always come with lambda expressions.   
In any other language like Python, closure are unrelated to Lambdas* 😊

A ***closure*** is any function that ***closes over*** the ***environment*** in which it was defined. This means that it can access variables not in its parameter list.

***
#### What is a Closure anyway in C++?

A value* defined (rather encapsulated) by lambda expression that consists of both the code as well as the values of the variables referred to in the code.  
(We will discuss this asterisk * further in this article)

So closure is an anonymous function object that is created automatically by the compiler as the result of a lambda expression. A closure stores those variables from the scope of the definition of the lambda expression that is used in the lambda expression.


> The runtime effect of a lambda expression is the generation of an object. Such objects are known as *closures*.

A closure is a function that encloses its surrounding state by referencing fields external to its body. The enclosed state remains across invocations of the closure

***
### Lambdas vs. Closures

S**cott Meyers** puts it beautifully — “**The distinction between a lambda and the corresponding closure is precisely equivalent to the distinction between a class and an instance of the class**”.


> Closures are to lambdas as objects are to classes.

As we know, a class exists only in source code; it doesn’t exist at runtime. What exists at runtime are objects of the class type. Similarily -


> Each lambda expression causes a unique class to be generated (during compilation) and also causes an object of that class type–a closure–to be created (at runtime).

1. Lambdas occupy no data memory at runtime, for example, though they may occupy code memory.
2. Closures occupy data memory, but not code memory.

***
### Distinction by Examples

Let’s take an example —

```cpp
auto f = [&](int x, int y) { return fudgeFactor * (x + y); };

// the expression to the right of the "=" is the lambda expression (i.e., "the lambda"), 

// The runtime object created by that expression is the closure.
```

`f` itself is not a closure, it is a ***copy of the closure*** 🥴

#### **Ok, what do you mean?**

The process of copying the closure into f may be optimized into a move but that doesn't change the fact that f itself is not the closure.

The actual closure object is a temporary that's typically destroyed at the end of the statement unless you bind it to a [forwarding reference(a.ka. Universal reference)](https://medium.com/pranayaggarwal25/universal-reference-perfect-forwarding-5664514cacf9) or lvalue-reference-to-`const`.

```cpp
//===============================================================//

auto&& rrefToClosure = [&](int x, int y) { return fudgeFactor * (x + y); };

const auto& lrefToConstToClosure = [&](int x, int y) { return fudgeFactor * (x + y); };

//===============================================================//
```
Let’s take another example —

```cpp
//===============================================================//

std::function<void(void)> closureWrapper1()
{
    int x = 10;
    return [&x](){ std::cout << "Value in the closure: " << x++ << std::endl; };
}
int main()
{
    int x = 10;
    auto func0 = [&x](){x += 1; std::cout << "Value in the closure: " << x << std::endl;};
    
    func0();  // Prints 11
    
    std::function<void(void)> func1 = closureWrapper1();  
    func1();  // Prints garbage value + 1 =~ garbage value
}

//===============================================================//
```
`func1` is not closure Instead, it’s a `std::function` wrapper object that wrapped a closure.

`func0` is a copy of a closure created by the lambda expression written after it.

***
### Close analog to a closure

**Function Object (Functor)** — Function object overload the operator(). It could capture the values by making a copy of the variables to its member variables. The shortcoming is that for each different function call, regardless of how simple it is, we would have to implement a new class, whereas implementing a lambda expression is faster.

***
### References

1. <http://scottmeyers.blogspot.com/2013/05/lambdas-vs-closures.html>
2. <https://leimao.github.io/blog/CPP-Closure/>
3. <https://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda>

***
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article. 

Please check out my [other articles](https://medium.com/pranayaggarwal25) and [website](http://pranayaggarwal.github.io/), Have a great day!

  