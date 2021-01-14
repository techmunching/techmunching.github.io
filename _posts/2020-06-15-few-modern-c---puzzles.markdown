---
layout:	post
title:	"Few Modern C++ Puzzles"
date:	2020-06-15
categories: [ 'C++', 'C++20' ]
image: img/0*3StcFi7Mii7eFJOZ.png
---

  Some puzzles from various talks, blog posts, and other bits

![](/img/0*3StcFi7Mii7eFJOZ.png)### Table of Contents

1. Puzzle 1: Capturing unique\_ptr by reference
2. Puzzle 2: Capturing the temporary by the reference
3. Puzzle 3: Making mistakes with std functions usage
4. Puzzle 4: Throwing away the dead
### Puzzle 1: Capturing unique\_ptr by reference

unique\_ptr<A> myFun()  
{  
 unique\_ptr<A> pa(new A());  
 return pa;  
}  
  
const A& rA = *myFun();This code compiles but rA contains garbage. Why is this code invalid?

**Note: **if we assign the return of myFun to a named unique\_ptr variable before dereferencing it, it works fine.

#### **Answer:**

The unique\_ptr will pass the ownership to another unique\_ptr, but in this code, there is nothing to capture the ownership from the returning pointer. In other words, It can not transfer the ownership, so it will be destructed. The proper way is:

unique\_ptr<A> rA = myFun(); // Pass the ownershipor

const A rA = *myFun(); // Store the values before destructionIn this code, the returning pointer will be destructed and the reference is referring to an object which is destructing soon after that using this reference invokes undefined behavior.

### Puzzle 2: Capturing the temporary by the reference

vector<bool> vb{true, true, false, true};auto proxy = vb[0];std::cout << proxy // **ok, Prints true**vb.reserve(100); // A: ???std::cout << proxy // **Error!!** likely to print false on clang, why?#### Answer:

Line A invalidates proxy so proxywas invalidated by vb.reserve.

Take below example —

void example\_1\_1\_3() {  
string\_view s; // s points to nullstring name = "abcdefghijklmnop";s = name; // A: s points to {name'} i.e. data owned by name’cout << s[0]; // B: ok – s[0] is ok because {a} is alivename = "frobozz"; // C: name modified => name’ is invalidcout << s[0]; // D: error – because it contains {invalid}}which is why guideline says —


> Never use the reference in case of a temporary argument.Take below example —

char& c = std::string{"hello my pretty long string"}[0];cout << c; // **(X) wrong** to initialize a  
// reference ‘c’ with an invalid pointer, pointer  
// was invalidated with temporary string was  
// destroyed.### Puzzle 3: Making mistakes with std functions usage

int main() {  
   
 auto x=10, y=2;  
 auto& good = min(x,y); // ok, {x,y}  
   
 cout << good; **// ok, fine.** auto& bad = min(x,y+1)  cout << bad; **// ERROR, why??**  
}#### Answer:

int main() {  
   
 auto x=10, y=2;  
 auto& good = min(x,y); // ok, {x,y}  
   
 cout << good; **// ok, fine.**auto& bad = min(x,y+1)   
 // A: IN: {x, temp(y+1)}  
 // OUT: temp2 obtained by {x,temp}// min() returns temp2  
  
 // temp destroyed hence → temp2 = {invalid}cout << bad; **// ERROR, bad initialized as invalid now**  
}In normal C++, this code compiles but has **undefined behavior**.  
Note In practice, on the three major compilers (GCC, VC++, clang) this code does not crash and appears to  
work. That’s because one manifestation of “undefined behavior” can be “*happens to do what you expect.*”

Nevertheless, this is undefined so one should be careful.

### Puzzle 4: Throwing away the dead

// godbolt.org/z/p\_QjCR  
static int gi = 0;  
void f() {  
 int i = 0;  
 throw &i; // ERROR, why??  
 throw &gi; // OK  
}#### Answer:

Unlike a return, the type of a thrown object cannot be carried through function signatures. Therefore, do not throw a Pointer with a lifetime other than static.

### References

1. <https://stackoverflow.com/questions/30858850/dereferencing-a-temporary-unique-ptr>
2. <http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1179r0.pdf>
  