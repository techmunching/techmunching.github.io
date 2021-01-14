---
layout:	post
title:	"Using custom deleters with shared_ptr and unique_ptr in C++"
date:	2020-05-29
categories: [ 'C++' ]
image: img/0*doMK1rnrcwhJmTks
author: admin
---

  How to use a custom deleter with an unique\_ptr and shared\_ptr

![](/img/0*doMK1rnrcwhJmTks)**Related post:** <https://medium.com/pranayaggarwal25/a-tale-of-two-allocations-f61aa0bf71fc>

### Table of Contents

1. Introduction
2. The true unknown face of smart pointers
3. What is std::default\_delete indeed?
4. Ways to specify custom deleters
5. Using custom deleter with shared\_ptr
6. Using custom deleter with unique\_ptr
7. Storage of custom deleters
8. Restrictions that come with custom deleters
### Introduction

Why and when would we need something like that?

**Case 1: **In order to fully delete an object sometimes, we need to do some additional action. What if performing “delete” (that smart pointers do automatically)is not the only thing which needs to be done before fully destroying the owned object.

**Case 2: **We can’t bind a shared\_ptr or unique\_ptr to a stack-allocated object, because calling delete on it would cause undefined behaviour.

**Case 3: **Mix of programming languages code, such as C++ with Obj-C++. As objective-c may need a complex release mechanism for its data types such as calling [CFRelease](https://developer.apple.com/documentation/corefoundation/1521153-cfrelease), we would be in need of a custom deleter.

**Case 4: **In C where, when you wrap FILE*, or some kind of a C style structure free(), custom deleters may be useful.

and a few other cases.

### The true unknown face of smart pointers

#### std::unique\_ptr

The complete type of ***std::unique\_ptr*** has a second template parameter, its deleter that has a default type std::default\_delete<T> .  
What is that?? No need to worry, We’ll cover this together :)

template< class T, **class Deleter = std::default\_delete<T>** >   
class unique\_ptr;  
// Manages a single objecttemplate < class T, **class Deleter**>   
class unique\_ptr<T[], Deleter>;  
// Manages a dynamically-allocated array of objects std::default\_delete<T> is a **function object** (a.k.a f**unctor**) that calls delete on the object when invoked. This is only the default type for invoking Deleter and it can be replaced with a custom deleter.

The invocation is done using operator() on the Deleter.

#### std::shared\_ptr

You can pass any callable thing (lambda, functor) as deleter while constructing a shared pointer in the constructor as an additional argument.

template< class Y, class Deleter >  
shared\_ptr( Y* ptr, Deleter d );// One of the overloads of shared\_ptr constructionthus specifying custom deleter with std::shared\_ptr is comparatively easy.

On ref count reaches zero, the shared\_ptr uses the [delete-expression](https://en.cppreference.com/w/cpp/language/delete "cpp/language/delete") i.e. delete ptr .

Also since C++17 —

// shared\_ptr can be used to manage a dynamically allocated array  
// since C++17 by specifying template argument with T[N] or T[]. So // you may writeshared\_ptr<int[]> myShared(new int[10]);### What is std::default\_delete indeed?

This is defined in <memory> header.

template< class T > struct default\_delete;template< class T > struct default\_delete<T[]>;1. The non-specialized default\_delete uses ***delete*** to deallocate memory for a single object.
2. A partial specialization for array types that uses ***delete[]*** is also provided.
#### Members:

1. Constructor — can be default or templated.
**constexpr default\_delete() noexcept = default; **// default**template <class U>  
default\_delete( const default\_delete<U>& d ) noexcept;** // templated  
// *Constructs a **std::default\_delete** object from another.  
// Overload resolution if **U*** is implicitly convertible to **T***.***2. operator() — **overload for the operator() is needed for the callability of struct/class as its a function object( or functor).  
At the point in the code where, this operator() is called, the type must be complete and defined.

Examples:

**Example 1:  
**{  
 std::unique\_ptr<int> ptr(new int(5));  
}   
// unique\_ptr<int> uses default\_delete<int>====================================================================**Example 2:  
**{  
 std::unique\_ptr<int[]> ptr(new int[10]);  
}   
// unique\_ptr<int[]> uses default\_delete<int[]>====================================================================**Example 3:**  
// default\_delete can be used anywhere a delete functor is needed  
std::vector<int*> v;for(int n = 0; n < 100; ++n)  
 v.push\_back(new int(n));std::for\_each(v.begin(), v.end(), **std::default\_delete<int>()**);  
// Constructing the function object to be called====================================================================**Example 4:**{  
 std::shared\_ptr<int> shared\_bad(new int[10]);   
}   
// the destructor calls delete, undefined behavior as it's an array  
   
{  
 std::shared\_ptr<int> shared\_good(new int[10], std::default\_delete<int[]> ());  
} // **the destructor calls delete[], ok**====================================================================  
**Example 5: (Valid only C++17 onwards)  
**{**  
**shared\_ptr<int[]> shared\_best(new int[10]);  
}  
// **the destructor calls delete[], awesome!!**====================================================================### Ways to specify custom deleters

1. std::function — Heavy size contribution ( ~32 bytes! on x64)
2. Function pointer — Just a pointer
3. Stateless functor / Stateless Lambda — None.
4. Stateful functor / Stateful Lambda — sizeof(functor or lambda)
### Using custom deleter with shared\_ptr

Examples —

#### 1. Use a proper functor —

(Requires custom deleter for array only Prior to C++17)

// declare the function objecttemplate< typename T >  
struct array\_deleter  
{  
 void operator ()( T const * p)  
 {   
 delete[] p;   
 }  
};// and use shared\_ptr as follows by constructing function object  
std::shared\_ptr<int> sp(new int[10], **array\_deleter<**int**>**());#### 2. Use a plain lambda

std::shared\_ptr<MyType> sp(new int[10], [](int *p) { delete[] p; });

#### 3. Use default\_delete (Only valid for array types before C++17)

std::shared\_ptr<int> sp(new int[10], std::default\_delete<int[]>());

**Note: **delete ptr is same as specifying default\_delete<T>{}ptr .

### Using custom deleter with unique\_ptr

With unique\_ptr there is a bit more complication. The main thing is that a deleter type will be part of unique\_ptr type.

By default we get std::default\_delete so here are some examples —

For a class ***MyType***

class MyType { // ...  
 // ...};void deleter(MyType*) {  
// ...  
}====================================================================  
**// 1. std::function   
**std::unique\_ptr<MyType, std::function<void (MyType*)>> u1(new MyType());OR std::unique\_ptr<MyType, decltype(deleter)>> u1(new MyType(), &deleter); **// 2nd argument is optional always as functor object is created by default**====================================================================**// 2. Function pointer**  
std::unique\_ptr<MyType, void (*)(MyType *)> u2(new MyType());====================================================================// A stateless functor  
struct MyTypeDeleterFunctor {   
 void operator()(MyType* p) {  
 // ...  
 }  
};**// 3. Stateless functor**  
std::unique\_ptr<MyType, MyTypeDeleterFunctor>u3(new MyType());====================================================================// 4.   
void close\_file([std::FILE](http://en.cppreference.com/w/cpp/io/c)* fp) { [std::fclose](http://en.cppreference.com/w/cpp/io/c/fclose)(fp); }### Storage of custom deleters

**For shared\_ptr  
**When you use a custom deleter it won’t affect the size of your shared\_ptr type. If you remember, shared\_ptr size should be roughly 2 x sizeof(ptr) so where does this deleter hide?

As we know, shared\_ptr consists of two things: pointer to the object and pointer to the control block (that contains ref count). Inside the control block structure of shared\_ptr, there is a space for custom deleter and allocator.

**For unique\_ptr  
**unique\_ptr is small and efficient; the size is one pointer so where is the custom allocator hide in this case?

The deleter is part of the type of unique\_ptr. And since the functor/lambda that is stateless, its type fully encodes everything there is to know about this without any size involvement. Using function pointer takes one pointer size and std::function takes even more size.


> The shared\_ptr always stores a deleter, this erases the type of the deleter, which might be useful in APIs. The disadvantages of using shared\_ptr over unique\_ptr include a higher memory cost for storing the deleter and a performance cost for maintaining the reference count.**Trivia:** The size of weak\_ptr is the same as that of shared\_ptr. Weak pointer points to the same control block as it’s shared pointer. When a weak\_ptr is created, destroyed, or copied a second reference count (weak pointer reference count) is manipulated. Weak count is connected with object storage deallocation (Refer prerequisite [talk](https://medium.com/pranayaggarwal25/a-tale-of-two-allocations-f61aa0bf71fc))

### Restrictions that come with custom deleter

#### Can’t use make\_shared with shared\_ptr

Unfortunately, you can pass a custom deleter only in the constructor of shared\_ptr there is no way to use make\_shared. This might be a bit of disadvantage (Refer prerequisite [talk](https://medium.com/pranayaggarwal25/a-tale-of-two-allocations-f61aa0bf71fc))

One can use [allocate\_shared](http://en.cppreference.com/w/cpp/memory/shared_ptr/allocate_shared) and custom allocator and deleter, but that’s too complex to be covered in this article.

#### Can’t use make\_unique with unique\_ptr

Similarly as with shared\_ptr you can pass a custom deleter only in the constructor of unique\_ptr and thus you cannot use make\_unique.

### References:

1. [https://www.reddit.com/r/cpp/comments/4gu77b/code\_and\_graphics\_custom\_deleters\_for\_c\_smart/](https://www.reddit.com/r/cpp/comments/4gu77b/code_and_graphics_custom_deleters_for_c_smart/)
2. <https://www.bfilipek.com/2016/04/custom-deleters-for-c-smart-pointers.html>
3. [https://en.cppreference.com/w/cpp/memory/default\_delete](https://en.cppreference.com/w/cpp/memory/default_delete)
4. <https://stackoverflow.com/questions/51255583/shouldnt-stdshared-ptr-use-stddefault-delete-by-default>
5. <https://www.programming-books.io/essential/cpp/using-custom-deleters-to-create-a-wrapper-to-a-c-interface-e8fe82bfeff74f699dc810b6cd5ce57a>
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out my [other articles](https://medium.com/pranayaggarwal25) and [website](http://pranayaggarwal.github.io/), Have a great day!

  