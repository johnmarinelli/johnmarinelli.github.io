---
title: C Struct Packing
updated: 2014-10-10
---

## Struct sizes in C

Here's an interesting little tidbit about structs and C: the size of the struct depends the order in which you declare your struct's member variables. This ancient art is called "struct packing". Let's look at an example:

```c
#include <stdio.h> 

struct X { int a; char b; int c; }; 

int main(void) { 
  /* note: %zu was added in C99 as a format specifier for size_t type */ 
  printf("%zu\n", sizeof(int)); // 4 
  printf("%zu\n", sizeof(char)); // 1 
  printf("%zu\n", sizeof(struct X)); // 12 
  return 0; 
}
```

This code makes sense - the size of two ints is 8, one char is 9, and the compiler pads our data to a multiple of 4 for efficiency. Fun fact: in gcc, you can use -fpack-struct to get `sizeof(struct X)` to return 9\. However, you probably don't want to do this unless you know *exactly* what you're doing, because although you save memory this way, the code will take much longer to execute. Moving on!

Let's add another `char` to our struct. Where shall we put it? Try appending it to the end:

```c
struct X { int a; char b; int c; char d; };
```

Compile this and run. The size of `struct X` should now return 16\. Why? Well, it turns out that the compiler will see the first three elements as being of size 12 already, and the extra `char d` adds 1, giving us 13, and 13 becomes 16\. Let's try putting `char d` immediately after `char b`:

```c
struct X { int a; char b; char d; int c; };
```

Compile and run. Whoa! `sizeof(struct X)` now returns 12! The compiler now sees the first three elements as being of size 8, and adds the int to return 12.

Moral of the story: place your data in memory-size order in your C structs! Other compilers do this automatically, but C/C++ don't.
