---
layout:	post
title:	"Coroutines and their introduction in C++"
date:	2020-05-30
categories: [ 'C++', 'C++20' ]
image: 0_qg0kAy5Zjb9DiXaU
tags: [ featured ]
author: admin
---

  Let’s discuss what coroutines are in general and how C++20 is introducing them

{% include pictures.html img="0_qg0kAy5Zjb9DiXaU" alt="what coroutines are in general" %}

***

### Table of Contents
1. Prerequisite Terminology  
2. What are coroutines?  
3. Coroutines vs Subroutines?  
4. Coroutines vs Threads  
5. Applications of coroutines  
6. Example in Python  
7. How to simulate coroutines in traditional C++  
8. Coroutines in C++20  
9. Restrictions

***

### Prerequisite Terminology

1. **Cooperative Multitasking (a.k.a non-preemptive multitasking)** — If multitasking participant process or thread voluntarily let go of control periodically or when idle or logically blocked. This type of multitasking is called *“cooperative”* because all programs must cooperate for the entire scheduling scheme to work.
2. **Subroutine** — Any regular function that you write is a subroutine.

***

### What are Coroutines?

**Coroutines** are stackless functions designed for enabling co-operative Multitasking, by allowing execution to be suspended and resumed.

**Coroutines** suspend execution by returning to the caller and the data that is required to resume execution is stored separately from the stack. This allows for sequential code that executes asynchronously (e.g. to handle non-blocking I/O without explicit callbacks), and also supports algorithms on lazy-computed infinite sequences and other uses.

This is why coroutines are well-suited for implementing familiar program components such as cooperative tasks, exceptions, event loops, state machines, and pipes.

***

### Coroutines vs Subroutines?

* With subroutines, execution begins at the start and finished on exit.
* Subroutines are special cases of coroutines. Any subroutine can be translated to a coroutine which does not call ‘yield’ (relinquish control).
* Subroutines only return once and don't hold the complete state between invocations.
{% include pictures.html img="0_u8LQUT-o3R32MBgP" alt="" %}

In contrast —

* Coroutines can exit by calling other coroutines, which may later return to the point where they were invoked in the original coroutine; from the coroutine’s point of view, it is actually not exiting but calling another coroutine.
* A coroutine instance holds state and varies between invocations.
{% include pictures.html img="1_bYDtHHWQDCvDt3XZAkNVoQ" alt="" %}

***

### Coroutines vs Threads

1. Coroutines are designed to be performing as lightweight threads.
2. **Coroutines provide concurrency but not parallelism [Important!]**
3. Switching between coroutines need not involve any system/blocking calls so no need for synchronization primitives such as mutexes, semaphores.

Thus coroutines —

1. provide asynchronicity and resource locking isn't needed.
2. are useful in functional programming techniques.
3. increase locality of reference.

***

### Applications of Coroutines

1. **Actor Model**: They are very useful to implement the actor model of concurrency. Each actor has its own procedures, but they give up control to the central scheduler, which executes them sequentially.
2. **Generators**: It is useful to implement generators that are targeted for streams particularly input/output and for traversal of data structures.
3. **Reverse Communication**: They are useful to implement reverse communication which is commonly used in mathematical software, wherein a procedure needs the using process to make a computation.


**Example 1—**To read a file and parse it while finding (matching) some meaningful data, you can either read step by step at each line, which is fine. You may also load the entire content in memory, which won’t be recommended for large text.

Coroutines are there to throw away the stack concept completely. Stop thinking of one process as the caller and the other as the callee, and start thinking of them as **cooperating equals**.

{% include pictures.html img="0*DaO1S9QPlS2HOrhA" alt="Execution flow for reading a file and finding text" %}

**Example 2 —**You have a consumer-producer relationship where one routine creates items and adds them to a queue and another removes items from the queue and uses them. For reasons of efficiency, you want to add and remove several items at once. The pseudo-code might look like this:
<pre>
<code>
    <i>var</i> q := new queue 

    <b>coroutine</b> produce  
        <b>loop</b>
            <b>while</b> q is not full  
                create some new items  
                add the items to q  
            <b>yield</b> to consume

    <b>coroutine</b> consume  
        <b>loop</b> 
            <b>while</b> q is not empty  
                remove some items from q  
                use the items  
            <b>yield</b> to produce

</code>
</pre>

 The queue is then completely filled or emptied before yielding control to the other coroutine using the *yield* command.

(This example is often used as an introduction to multithreading, two threads are not a must need for this).

***

### An example in Python

If you have used Python, you may know that there is a keyword called yield that allows loop back and forth between the caller and the called function until the caller is not done with function or the function terminates because of some logic it is given.

```python

# A Python program to generate numbers in a range using yield
def rangeN(a, b):
  i = a 
  while (i < b):
    yield i
    i += 1  # Next execution resumes from this point
for i in rangeN(1, 5):
  print(i)
```

***

### How to simulate coroutines in traditional C++

To simulate coroutines in traditional C++ is challenging as for every response to a function call, there is a stack being initialized that keeps track of all its variables and constants and gets destroyed when the function call ends.

For the same range example, to simulate a simple switch coroutine suspend-resume we can do something like —
```cpp
// A bad simulation of coroutine, no state saving  
#include<iostream>
int range(int a, int b)   
{   
  static long long int i = a-1;  
  for (;i < b;)   
  {   
    return ++i;   
  }   
  return 0;   
}

int main()   
{   
  int i; 
  for (; i=range(1, 5);)   
    std::cout << i << '\n';  
  
  return 0;   
}
```
However, this doesn’t hold good for coroutines criteria of saving/resuming from the saved state :(

```cpp
// A better simulation of coroutine, state saving!!  
#include<iostream>int range(int a, int b)

{
  static long long int i;
  static int state = 0;
  switch (state) {
  case 0:
    /* start of function */
    state = 1;
    for (i = a; i < b; i++) {
      return i; /* Returns control */

      case 1: ; /* resume control straight after the return */
    }
  }
  state = 0;
  return 0;
}

int main() {
  int i;
  for (; i = range(1, 5);)
    std::cout << i << '\n';
  return 0;
}
```

***

### Coroutines in C++20

In c++20, coroutines are coming. A function is a coroutine if its definition does any of the following:

* uses the *co\_await* operator to suspend execution until resumed.
* uses the keyword *co\_yield* to suspend execution returning a value.
* uses the keyword *co\_return* to complete execution.
Let’s take a similar example to get a range.

For the simplicity of this post, let’s assume a generator template is something that exists already and can be used to generate a range,

(This blog post from Microsoft [*https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/october/c-from-algorithms-to-coroutines-in-c*](https://docs.microsoft.com/en-us/archive/msdn-magazine/2017/october/c-from-algorithms-to-coroutines-in-c) is amazing regarding the generator pattern details)

```cpp
#include <iostream>

#include <vector> // Coroutine gets called on need

generator <int> generateNumbers(int begin, int inc = 1) {

  for (int i = begin;; i += inc) {
    co_yield i;
  }

}

int main() {

  std::cout << std::endl;

  const auto numbers = generateNumbers(-10);

  for (int i = 1; i<= 20; ++i)
    std::cout << numbers << " "; // Runs finite = 20 times***  

  for (auto n:generateNumbers(0, 5)) // Runs infinite times**  
    std::cout << n << " "; // (3)  

  std::cout << "\n\n";

}
```

We’ll cover more about coroutines later as it gets better documented and evolved.

***

### Restrictions

Every coroutine in C++ has some restrictions noted below. So coroutines —

1. **Can’t return** with [variadic arguments](https://en.cppreference.com/w/cpp/language/variadic_arguments "cpp/language/variadic arguments")
2. **Can’t return** using plain *return*
3. **Can’t return** [placeholder](https://en.cppreference.com/w/cpp/language/function "cpp/language/function") (*auto* or *Concept*)
4. **Can’t be** *constexpr* functions.
5. **Can’t be** constructors or destructors.
6. **Can’t be** the main function.

***

#### References

1. <https://en.cppreference.com/w/cpp/language/coroutines>
2. <https://www.xenonstack.com/insights/coroutines/>
3. <https://stackoverflow.com/questions/48106252/why-threads-are-showing-better-performance-than-coroutines>
4. <https://www.modernescpp.com/index.php/c-20-coroutines-the-first-overview>
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve!!  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com), Have a great day!

  