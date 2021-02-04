---
layout:	post
title:	"Move Semantics in C++ : Part 1"
date:	2020-03-22
categories: [ 'C++', 'Interview Preparation' ]
image: 'img/0_TbvZrhoMfgtPAsaa.webp'
author: admin
---

  **Prerequisites**: [Understanding of rvalue references](https://medium.com/pranayaggarwal25/rvalue-references-e99dfd3933ff)

![Move Semantics](/img/0_TbvZrhoMfgtPAsaa.webp)

***

### What are rvalues?


> RValue references represent modifiable object where the value is no longer needed so that you can steal their content and provide move semantics.

***

### Table of Contents

1. Where does move semantics work?
2. The difference in C++03/C++98 vs C++11
3. How to enable Move Semantics
4. Default C++ support for Move Semantics
5. How to forward Move Semantics
6. Perfect Forwarding
7. Move semantics and default Template Arguments

***

### Where does Move Semantics work?

```cpp
std::vector createAndInsert() {

  std::vector retval;
  retval.reserve(3);
  std::string val("temp");

  // Step 1  
  retval.push_back(val);
  // C++98/03 : insert a copy of s into coll  
  // C++11. : same 

  // Step 2  
  retval.push_back(val + val);
  // C++98/03 : insert copy of temp val+val, destroy temp value  
  // C++11. : move val+val 

  // Step 3  
  retval.push_back(std::move(val));
  // move explicitly

  // Step 4  
  return retval;
  // C++98/03 : may copy call  
  // C++11. : may move call  
}

std::vector vec;   
vec = createAndInsert();

```

![](/img/1_Skf__7ocaqDjgy0NuzonpQ.webp)*Final Memory footprint after createAndInsert() function call*

***

### The difference in C++03/C++98 vs C++11

Containers have value semantics i.e. copy passed new elements into their containers but they allow to pass values which lead to unnecessary copies with C++98/C++03.

As a contrast, with rvalue references, you can provide move semantics in C++11. For e.g. following has been added in vector class in C++11 —

```cpp
template class vector {
  public: 
  
    ...

    // insert a copy of elem:   
    void push_back(const T & elem);

    ... 
    
    // insert elem with its content moved: // Introduced in C++11
    void push_back(T && elem);

    ...
};
```
***

### How to enable Move Semantics


> To support move semantics for non-trivial types you should allow to  
> - **Steal** contents from the passed object  
> - **Set** the assigned object in a valid but undefined (or initial) state.

by providing a **move constructor** and a **move assignment operator**.

A sample move constructor for string class is as below —
```cpp
class string {
  private:
    int len; // current number of characters   
  char * elems; // array of characters  
  
  public:  
  
  // create a full copy of s:  
  string(const string & s): len(s.len) {
    elems = new char[len + 1]; // new memory  
    memcpy(elems, s.elems, len + 1);
  }

  // Move Constructor  
  // create a copy of s with its content moved:   
  string(string && s): len(s.len), elems(s.elems) {
      // copy pointer to memory  
      s.elems = nullptr;
      // otherwise destructor of s frees stolen memory   

      s.len = 0;
    }

    ...
};
```
However, primitive types don’t get any benefit out of move

```cpp
class cannot_benefit_from_move_semantics  
{  
 int a; // moving an int means copying an int  
 float b; // moving a float means copying a float  
 double c; // moving a double means copying a double  
 char d[64]; // moving a char array means copying a char array  
  
 // ...  
};
```
***

### Default C++ support for Move Semantics

1. **Default Move Support —** For library objects “Unless otherwise specified, moved-from objects shall be placed in a valid but unspecified state.”.
2. **Copy as Fallback -** If no move semantics is provided, copy semantics is used.
3. **Move is very much needed** — If move constructor is declared as deleted, the program may be ill-formed (not in C++17)
4. **Default move operations** — Move constructor and Move assignment operator are generated only if there is no special member function defined among —   
• Copy constructor   
• Assignment operator   
• Destructor

***

### How to forward Move Semantics

forwarding needs to done explicitly
```cpp
===========================================================

class X;

void g (X&); // for variable values   
void g (const X&); // for constant values   
void g (X&&); // for values that are no longer used (move semantics) 

===========================================================

void f(X & t) {
  g(t); // t is non const lvalue => calls g(X&)   
}

void f(const X & t) {
  g(t); // t is const lvalue => calls g(const X&)   
}

void f(X && t) {
  g(std::move(t)); //non-const lvalue needs ::move(), calls g(X&&)   
}

===========================================================
```
**Examples**
```cpp
===========================================================
X v;
const X c;

1. f(v); // calls f(X&) => calls g(X&)   
2. f(c); // calls f(const X&) => calls g(const X&)   
3. f(X()); // calls f(X&&) => calls g(X&&)   
4. f(std::move(v)); // calls f(X&&) => calls g(X&&)  
===========================================================
```
***

### Perfect forwarding

Special semantics for **&& with template types**, are known as “***Universal Reference***” (standard term known as “***Forwarding Reference***”)


> You can use *std::forward()* to keep this forwarding semantics

```cpp
===========================================================
class X;

void g(X & ); // for variable values   
void g(const X & ); // for constant values   
void g(X && ); // for values that are no longer used (move semantics)

template < typename T >
  void f(T && t) // t is universal/forwarding reference 
{
  g(std::forward(t)); // forwards move semantics   
  // (without forward, only calls g(const X&) or g(X&))  
}

===========================================================

X v;   
const X c; 

f(v);  
f(c);   
f(X());   
f(std::move(v)); // All f(..) call work fine

```
***

### Move semantics and default Template Arguments

Now the question would be why can’t we just use directly universal reference in our class copy/move constructors. The reason behind that is once Type deduction has taken place, “&&” even though templated, is no longer a universal reference. So basically
```cpp
template <typename T>  
void f (T&& t) // t is universal/forwarding reference 
{   
...  
}

===========================================================

template<typename T>  
class Widget {  
 ...  
 Widget(Widget&& rhs);   
 // fully specified parameter type ⇒ no type deduction  
 // hence && ≡ rvalue reference here};
}
 
 ===========================================================  
   
template<typename T1>  
class Gadget {  
 ...  
 template<typename T2>  
 Gadget(T2&& rhs);   
 // deduced parameter type ⇒ type deduction;  
 // hence && ≡ universal reference};
}
 
 ===========================================================  
```
This is why all params can’t be used to be forwarded easily as that requires multiple definitions for each argument to de deduced separately and further need default call + default template arguments.

Another issue with default template arguments is that they are better match than pre-defined copy constructor for non-const objects —

**Using forwarding references only - The problem**
```cpp

class Cust {
  private:

    std::string first;
    std::string last;
    long id;

  public:

    //better match than pre-defined copy constructor for non-const objects  
    template < typename STR1, typename STR2 = std::string >
    Cust(STR1 && fn, STR2 && ln = "", long i = 0): 
      first(std::forward < STR1 > (fn)),
      last(std::forward < STR2 > (ln)),
      id(i) {}
      
      ...
};
 
std::vector v;   
v.push_back(Cust("Tim","Coe",42)); // OK

Cust c("Joe","Fix",77);   
v.push_back(c); // OK

Cust d1{"Tina"};        // OK   
const Cust d2{"Bill"};  // OK   
Cust e1{d1};            // Error:can't convert Cust to string 
Cust e2{d2};            // OK

```
If member templates can be used as copy/move constructor or assignment operator, overload the first argument instead of using a template parameter, which is an easier and performant way to have move semantics.

**Using forwarding references, a performant way**

```cpp

class Cust {

  private:
    std::string first;
    std::string last;
    long id;

  public:

    template < typename STR2 = std::string >
    Cust(const std::string & fn, STR2 && ln = "", long i = 0):
        first(fn), last(std::forward(ln)), id(i) {}
    
    template < typename STR2 = std::string >
    Cust(std::string && fn, STR2 && ln = "", long i = 0):
        first(std::move(fn)), last(std::forward(ln)), id(i) {}
    
    ...
};

std::vector v;   
v.push_back(Cust("Tim","Coe",42)); // OK 

Cust c("Joe","Fix",77);   
v.push_back(c); // OK 

Cust d1{"Tina"}; // OK   
const Cust d2{"Bill"}; // OK   
Cust e1{d1}; // OK   
Cust e2{d2}; // OK
```

Another safe way can be

```cpp
// Safe but non-performant way  
class Cust {   
   
  Cust(std::string fn, std::string ln = “”, long i = 0) :   
    first(std::move(fn)), last(std::move(ln)), id(i)   
    {   
      // ...
    }   
};
```
***

### References

1. Modern C++ — [Dreams and Nightmares](http://www.josuttis.com/talks/Josuttis_MoveNightmare_170517.pdf) by Nicolai M. Josuttis
2. [what-is-move-semantics](https://stackoverflow.com/questions/3106110/what-is-move-semantics)
  