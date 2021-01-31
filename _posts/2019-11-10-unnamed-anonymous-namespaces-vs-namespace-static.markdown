---
layout:	post
title:	"Unnamed anonymous namespaces vs namespace static"
date:	2019-11-10
categories: [ 'Git' ]
image: img/0*Z2Qy5Q3HZhjjy6Bx
author: admin
---

  Detailed Comparison of unnamed namespaces and namespace static with advantages and disadvantages   ![](/img/0*Z2Qy5Q3HZhjjy6Bx)Photo by [Jens Lelie](https://unsplash.com/@leliejens?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)### Table of Contents

1. Translation unit  
2. unnamed namespaces  
3. static and namespace-static  
4. usage for both  
5. Where anonymous namespaces are superior  
6. Where namespace static pays off  
7. Summary

### Translation Unit

According to [standard C++](http://www.efnetcpp.org/wiki/ISO/IEC_14882) ([wayback machine link](http://web.archive.org/web/20070403232333/http://www.efnetcpp.org/wiki/ISO/IEC_14882))


> *A translation unit is the basic unit of compilation in C++. It consists of the contents of a single source file, plus the contents of any header files directly or indirectly included by it, minus those lines that were ignored using conditional preprocessing statements.*### Unnamed namespace

A feature of C++ is the ability to create unnamed (anonymous) namespaces, like so:

namespace {  
 int invisible\_to\_others\_translation\_unit() { ... }} // namespaceThese unnamed namespaces *are* accessible within the translation unit they’re created in as if you had an implicit using-clause to them. An [*unnamed-namespace-definition*](http://eel.is/c++draft/namespace.def#nt:unnamed-namespace-definition) behaves as if it were replaced by

*inline namespace unique *{* /* empty body */ *}*  
using namespace unique *;*  
namespace unique *{* *[*namespace-body*](http://eel.is/c++draft/namespace.def#nt:namespace-body)* *}
> Members of an inline namespace are treated as if they are members of the enclosing namespace.### Static and namespace-static

The **static** keyword can be used to declare variables and functions at -

1. global scope — variables and functions
2. namespace scope — variables and functions
3. class scope — variables and functions
4. local scope — variables
Static duration means that the object or variable is allocated when the program starts and is deallocated when the program ends.

**External linkage** refers to things that exist beyond a particular translation unit. In other words, accessible through the whole program.

**Internal linkage** means everything in the scope of a translation unit.

By default, an object or variable that is defined in the global namespace has the static duration and **external linkage** however when you declare a variable or function at file scope (global and/or namespace scope), the **static** keyword specifies that the variable or function has **internal linkage**.

// **in namespace or global scope**int i; // extern by default  
const int ci; // static by default  
extern const int eci; // explicitly extern  
static int si; // explicitly static  
**// same goes for functions (but there are no global const functions)**int foo(); // extern by default  
static int bar(); // explicitly static
> *const** variables goes by default internal linkage unless otherwise declared as **extern**.*1. Static data members/member function/unnamed class/named class/enumeration of a class in namespace scope have same linkage as the enclosing class.
2. A class template has the linkage of the innermost enclosing class or namespace in which it is declared.
3. Specializations (explicit or implicit) of a template that has internal linkage are distinct from all specializations in other translation units.
### Static usage vs anonymous namespace

namespace test{  
 static int i = 5; // internal linkage, definition is here  
 // And there will be no multiple definition!  
}if you remove static, multiple definition errors would surely come.

vs

namespace {  
 int i = 5; // default internal linkage, unreachable from other   
 // translation units.  
 class invisible\_to\_others { };  
}### Where anonymous namespaces are superior

#### 1. namespaces work for everything

An unnamed namespace is superior to the static keyword, primarily because of the fact that keyword static applies only to the *variables* declarations and functions, not to the user-defined *types*.

The following code is valid in C++

 //legal code  
 static int sample\_function() { /* function body */ }  
 static int sample\_variable;But this code is NOT valid

//illegal code  
 static class sample\_class { /* class body */ };  
 static struct sample\_struct { /* struct body */ };So the solution is, unnamed-namespace, which is this,

 //legal code  
 namespace   
 {   
 class sample\_class { /* class body */ };  
 struct sample\_struct { /* struct body */ };  
 }#### 2. static has too many meanings

a) namespace-static — internal linkage  
b) local-static — local variable persistence  
c) member-static — class method

#### 3. Uniformity and consistency

Namespaces provide a uniform and consistent way of controlling visibility at the global scope. You don’t have to use different tools for the same thing.

#### 4. For template arguments

C++ does not allow types and pointers/references to objects or functions with internal linkage (static) to be used as template parameters.

### Where static in namespaces pays off

#### 1. Specialization of templates


> anonymous namespaces can’t specialize templates outside of the namespace blockstatic keyword makes every translation unit which includes the interface header that contains namespace declaration has an internal-linkage copy of the static variable/function (consumes space). Hence, no multiple definition error. The same for const with no extern preceding it which defines a variable of internal linkage.

#### 2. A cleaner Global Symbol Table

static prevent the name from entering into the global symbol table. This is strictly an optimization, but an important one in practice. This property is not shared by names in the unnamed namespace.


> Functions in unnamed namespaces, however, generally are added to the symbol table but marked as internal or mangled so that they are effectively inaccessible.So, in general, a program that uses static for its translation-unit-local namespace-level functions generates less work for the linker and might execute faster than the equivalent program using the unnamed namespace.

![](/img/0*EHATWzKSD1maPiFs.gif)### Summary

Avoid working with anonymous namespaces if you’re working with headers. Due to default internal linkage, each translation unit will define its own unique instance of members of the unnamed namespace which can cause unexpected results, bloat the resulting executable, or inadvertently trigger [undefined behavior](https://wiki.sei.cmu.edu/confluence/display/cplusplus/BB.+Definitions#BB.Definitions-undefinedbehavior) due to one-definition rule (ODR) violations.

Stating that variables defined in the anonymous namespaces are implicitly static is **wrong** and static are **not** totally useless in namespaces. In general, you can go to this guideline.


> Prefer free functions and types in anonymous namespace (unless in header files, then go for static).
> If needed to specialize class template methods or linker optimizations are a major concern, static is the option to go for.Unnamed namespaces are designed for protecting locality rather than offering an interface.

### Bonus Question 1

namespace { class A { } ; }  
  
void foo ( A ) // \_Z3fooN12\_GLOBAL\_\_N\_11AE  
{ ; }the function's symbol, apparently, would refer to the name of A, which is a member of a uniquely named namespace.

what is the foo's linkage.?

#### Answer

While the name foo technically has external linkage, it cannot actually be referred to by other translation units since there is no way to write the name of the type of foo's parameter

### References

[**linkage of function with parameter from unnamed namespace**  
*Thanks for contributing an answer to Stack Overflow! Please be sure to answer the question. Provide details and share…*stackoverflow.com](https://stackoverflow.com/questions/41625435/ "https://stackoverflow.com/questions/41625435/")[](https://stackoverflow.com/questions/41625435/)[**DCL59-CPP. Do not define an unnamed namespace in a header file**  
*Unnamed namespaces are used to define a namespace that is unique to the translation unit, where the names contained…*wiki.sei.cmu.edu](https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL59-CPP.+Do+not+define+an+unnamed+namespace+in+a+header+file "https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL59-CPP.+Do+not+define+an+unnamed+namespace+in+a+header+file")[](https://wiki.sei.cmu.edu/confluence/display/cplusplus/DCL59-CPP.+Do+not+define+an+unnamed+namespace+in+a+header+file)Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com), Have a great day!

  