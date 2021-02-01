---
layout:	post
title:	"Find the element that appears once in array but other elements appear thrice"
date:	2020-01-26
categories: [ 'Algorithm', 'Interview Preparation' ]
image: 'img/0*CYH--jxfZ8bQGcXj.png'
tags: [ featured ]
author: admin
---

  ![](/img/0*CYH--jxfZ8bQGcXj.png)
  
***

### Problem

Given an array that contains integers. The content is such that every integer occurs 3 times in that array leaving one integer that appears only once.  
Find out the fastest way to find that single integer  
 — not using any auxiliary memory.

***

### Example

*Input: arr[] = {12, 1, 12, 3, 12, 1, 1, 2, 3, 3}  
Output: 2  
In the given array all element appear three times except **‘2'** which appears once.*

### XOR Thoughts

I know what you’re thinking. Seems fine, we’d do some XOR stuff and get the answer. Um.. yes but not that straightforwardly.

Had the problem been simple such as —


> “Finding the element which appears once in an array — containing other elements each appearing twice”.

Solution was to XOR all the elements and you get the answer. Basically, it makes use of the fact that x^x = 0. So all paired elements get XOR’d and vanish leaving the lonely element.

How you ask — Well bitwise XOR is —

1. Associative — a ^ b ^ c = (a ^ b)^c  =  a ^(b ^ c)
2. Commutative — Irrespective of what fashion elements appear in the array.

***

### Trick

Now, in the current question — if we apply the above idea, it will not work because — we got to have every unique element appearing even number of times. So instead of getting the answer, we will end up getting XOR of all unique elements which isn’t exactly what we want.

To rectify this problem, the solution makes use of 2 variables.  
1) **ones** — At any point in time, this variable holds XOR of all the elements which have appeared **only once**.  
2) **twos** — At any point in time, this variable holds XOR of all the elements which have appeared **only twice**.

So if at any point time,  
1. A new number appears — It gets **XOR’d** to the variable “**ones**”.  
2. A number gets repeated(appears twice) — It is **removed** from “**ones**” and **XOR’d** to the variable “**twos**”.  
3. A number appears for the third time — It gets **removed** from both “**ones**” and “**twice**”.

The final answer we want is the value present in “ones” — because, it holds the unique element.

***

### Final Solution and explanation

Code for the solution in basic C++ would go something like this —

```cpp

int main() {
  int B[] = {
    1,
    1,
    1,
    3,
    3,
    3,
    20,
    4,
    4,
    4
  };
  int ones = 0;
  int twos = 0;
  int not_threes;
  int x;

  for (i = 0; i < 10; i++) {
    x = B[i];
    twos |= ones & x; // Step 1
    ones ^= x; // Step 2  
    not_threes = ~(ones & twos); // Step 3  
    ones &= not_threes; // Step 4  
    twos &= not_threes; // Step 5  
  }

  printf("\n unique element = %d \n", ones);
  return 0;

}

```
#### Step 1 and Step 2: New Element

Let's say a new element(x) appears — “**ones**” and “**twos**” haven’t recorded “x”.

```

  twos| = ones & x

```

AND condition yields nothing. So “**twos**” doesn’t get bit representation of “x”.  
But the next step ends up adding bits of “x” in “**ones**”.

```

  ones ^= x

```
#### Step 1 and Step 2: Second Time Element

Let's say an element(x) appears the second time. Now “**ones**” has recorded “x” but not “**twos**”.

```

  twos| = ones & x

```

“**twos**” ends up getting bits of x. But due to the statement,

```

  ones ^= x

```
“**ones**” removes “x” from its binary representation.

#### Step 1 and Step 2: Third Time Element

Let’s say an element(x) appears for the third time.  
Then “**ones**” does not have bit representation of “x” but “**twos**” has.

“ones & x” yield nothing .. “twos” by itself has bit representation of “x”. So after this statement, “two” has bit representation of “x”.  
Due to

```

  ones^=x

```
after this step, “one” also ends up getting bit representation of “x”.

#### Step 3, 4, and 5:

The last 3 lines of code convert common 1’s between “ones” and “twos” to zeros. So if an element has appeared once or twice, the last three lines do nothing.

If an element appears third time, last 3 lines of code remove common 1’s of “**ones**” and “**twos**” — which is the bit representation of “x”.  
Thus both “**ones**” and “**twos**” end up losing bit representation of “x”.

The final answer we want is the value present in “**ones**” — because, it holds the unique element.

***

### A dry run of the solution

```
  1st example  
  — — — — — —   
  2, 2, 2, 4

  After first iteration,  
  ones = 2, twos = 0  
  After second iteration,  
  ones = 0, twos = 2  
  After third iteration,  
  ones = 0, twos = 0  
  After fourth iteration,  
  ones = 4, twos = 0

  2nd example  
  — — — — — —   
  4, 2, 2, 2

  After first iteration,  
  ones = 4, twos = 0  
  After second iteration,  
  ones = 6, twos = 0  
  After third iteration,  
  ones = 4, twos = 2  
  After fourth iteration,  
  ones = 4, twos = 0
```
***

### References:

1. [Careercup](https://www.careercup.com/question?id=7902674)

  