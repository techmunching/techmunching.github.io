---
layout:	post
title:	"Using Modern C++ class members and initializations the right way"
date:	2020-07-03
categories: [ 'C++' ]
image: img/0*csRd0YjayeNTl96F.jpg
author: admin
---

  Using In-member initialization, using constructors smartly and using class members functions in a safe and proper way to avoid mistakes

![](/img/0*csRd0YjayeNTl96F.jpg)Clean code!### Table of Contents

1. Use member initializers in the same order as their declaration
2. Prefer in-class member initializer over constant initializations OR over default constructor.
3. Don’t cast away const, ever!
4. Use delegating constructors to represent common actions for all constructors of a class.
### 1. Use member initializers in the same order as their declaration

Member variables are always initialized in the order they are declared in the class definition.

**The order in which you write them in the constructor initialization list is ignored 🥴**

Make sure the constructor code doesn’t confusingly specify different orders. For e.g. this case as below —

![](/img/1*Wzlu5I7J1KamJhQYob7_4w.png)Would lead to issuesemail is declared before first\_name and last\_name in the class definition, hence as per the constructor call, it will be initialized first and will attempt to use the other not-yet-initialized fields which are first\_name and last\_name .

#### How to make it right

This code harbors a bug that’s as subtly harmful as it is hard to spot hence


> Write member initializers in the same order as their declarationMany compilers (but not all) will issue a warning if you break this rule. Modern compilers Clang, MSVC detect it with the right use of right warning flags.

#### Reason

The reason for this language design decision is to ensure there is a unique order to destroy members; otherwise, the destructor would have to destroy objects in different orders, depending on the constructor that built the object.

#### Benefit

1. Protects you from an oddity of the language without requiring everyone to know it.
2. Might encourage you to rethink your class design so this dependency goes away
### 2. Prefer in-class member initializer over constant initializations OR over default constructor

You should don’t define a default constructor that only initializes data members; use in-class member initializers instead which works as a good fallback in case you forget to initialize something.

Example — A bad class that misses one initialization in a constructor

![](/img/1*iRhyfPkavdlODmNqmNEKsw.png)**Cons of not using in-member class initializers**where the following is an example of a much better class

![](/img/1*GNGSOMx_9NcelulDaYnytg.png)**Using in-member class initializers, Nice!!**#### Reason

Using in-class member initializers lets the compiler generate the function for you. Also, the compiler-generated function can be more efficient 😊

#### Benefits

1. No overhead of taking care of initializing constants separately in each constructor.
2. Performance gain by using standard default constructors.
### 3. Don't cast away const, ever!

We shouldn’t cast away from getter functions even when there seems a need.

For e.g. — Stuff is a class that does some calculations overnumber1 and number2 and computes the result. Now getValue() const is a function that fetches the value, and being a getter function is marked const.

number1 and number2 are updated byService1() and Service2() functions respectively.

![](/img/1*AuUxrjVaYzHmo_OacXV5qA.png)Now, in case read frequency of getValue() is much more than the number of writes, we should preemptively update the cachedValue which is returned.

Such as —

![](/img/1*rqMbsx6k1igbaXt3OKW5dw.png)However, in case the number of writes is much more, we should follow **a lazy calculation approach** where we set a dirty flag such as below —

![](/img/1*TD_ZSa_Y2-jmsOZHZO8B2w.png)getValue function would show error as it’s marked constBut this poses a problem because const function can not modify this **newly introduced class member variable **cachedValid .

1. **A wrong fix **would be to remove const from** **getValue() function
2. **Another wrong fix **would be to const\_cast over “*this”* pointer.
#### Reason

Doing this makes a lie out of const. Any variable is actually declared asconst, modifying it may result in undefined behavior.

1. Allows getValue() function to change anything in the instance.
2. The header file is now speaking a lie basically.
#### Correct Fix

The right fix would be to declare cachedValid and cachedValue as mutable so that thegetValue() function can only modify the mutable ones.

![](/img/1*DgxaGoDSuAcpP58716AYRQ.png)The correct fix#### Benefits of correct fix

* Header file tells the truth
* getValue() function can only change the mutable variables
* Code accessing mutable members is shorter and more readable
* Easier to write, read, and maintain
* Const-correctness may enable optimizations 😊
### 4. Use delegating constructors to represent common actions for all constructors of a class

The common action gets tedious to write and may accidentally not be common. Hence, wherever possible we should refer to existing constructors.

For e.g. — This Date is a bad class.

![](/img/1*WNgLgmo1n2TEsdoGYPy_EQ.png)A bad series of constructors, duplicate logic![](/img/1*B5w6vP4oil0rnbYdMWkyng.png)**Good!! Using delegating constructors**#### Reason

To avoid repetition and accidental differences.

### References

1. <https://www.youtube.com/watch?v=XkDEzfpdcSg>
2. [C++ Coding Standards: 101 Rules, Guidelines, and Best Practices by Herb Sutter, Andrei Alexandrescu](https://www.amazon.com/Coding-Standards-Rules-Guidelines-Practices/dp/0321113586)
3. <https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines>
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out my [other articles](https://medium.com/pranayaggarwal25) and [website](http://pranayaggarwal.github.io/), Have a great day!

  