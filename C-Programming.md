# Want a quick introduction to C?
* Keep reading for the quick crash-course to C Programming below
* Then see the [[C Gotchas wiki page|C-Programming---Common-Gotchas]].
* Kick back relax with [Lawrence's intro videos](http://angrave.github.io/sysassets)

# External resources
* [C for C++/Java Programmers](http://www.ccs.neu.edu/course/com3620/parent/C-for-Java-C++/c-for-c++-alt.html)
* [C Tutorial by Brian Kernighan](http://www.lysator.liu.se/c/bwk-tutor.html)
* [c faq](http://c-faq.com/)
* Add your favorite resources here


# Crash Course Intro to C

*Warning new page* Please fix typos and formatting mistakes for me and add useful links too.*

## How do you write a complete hello world program in C?
```C
#include <stdio.h>
int main() { 
  printf("Hello World\n");
  return 0; 
}
```
## Why do we #include stdio.h?
We're lazy! We don't want declare the printf function. It's already done for us inside the file 'stdio.h'. The #include includes the text of the file as part of our file to be compiled.

## How are C strings represented?
Just characters in memory. The end of the string includes a NUL (0) byte. So "ABC" requires four(4) bytes. The only way to find out how long a C string is, is to keep reading memory until you find the nul byte.

## How do you declare a pointer? 
Pointers refer to memory address. The type of the pointer is useful - it tells the compiler how many bytes need to be read/written.
```
int* ptr1;
char* ptr2;
```

## How do you use a pointer to read/write some memory?
if 'p' is a pointer then user "*p" to write to the memory location(s) pointed to by p.
*ptr = 0; // Writes some memory. The number of bytes written depends on the pointer type.

## What is pointer arithmetic?
You can add an integer to a pointer. However the pointer type is used to determine how much to increment the pointer. For char pointers this is trivial because characters are always one byte:
```C
char *ptr = "Hello"; // ptr holds the memory location of 'H'
ptr +=2; ptr now points to the 'l"
```

If an int is 4 bytes then ptr+1 points to 4 bytes after whatever ptr is pointing at.

## What is a void pointer?
A pointer without a type. You can think of this as raw pointer, or just a memory address. You can directly read or write to it because the void type does not have a size.


## Does printf call write or does write call printf?
Ans: printf calls write. Printf includes an internal buffer so, to increase performance printf may not call write everytime you call printf. printf is a C library function. write is a system call.

## How do you print out pointer values? integers? strings?
Use format specifiers "%p" for pointers, "%d" for integers and "%s" for Strings.
(Todo: add link for more information / more examples here) 

## How would you make standard out be saved to a file?
Simplest way: run your program and use shell redirection
e.g.
```
./program > output.txt

#To read the contents of the file,
cat output.txt
```
More complicated way: close(1) and then use open to re-open the file descriptor.
See [[http://angrave.github.io/sysassets/chapter1.html]]
## What's the difference between a and b? Give an example of something you can do with a but not b
char a[] = "Hello";
char* b = "Hello";


## sizeof() returns the number of bytes. So using above code, what is sizeof(a) and sizeof(b)?
sizeof(a): a is an array. Returns the numbef of bytes required for the entire array (6)
sizeof(b): Same as sizeof(char*). Returns the number bytes required for a pointer (e.g. 4 or 8)

## Which of the following code is incorrect or correct and why?
```
int* f1(int *p) {
    *p = 42;
    return p;
} // This code is correct

char* f2() {
    char p[] = "Hello";
    return p;
} // Incorrect!
// An array p is created on the stack for the correct size to hold 
// H,e,l,l,o, and a null byte i.e. (6) bytes.
// This array is stored on the stack and is invalid after we return from s2.

char* f3() {
    char* p = "Hello";
    return p;
} // OK
// p is a pointer. It holds the address of the string constant. The string constant is unchange and valid even after f3 returns.

char* f4() {
    static char p[] = "Hello";
    return p;
} // OK
The array is static meaning it exists for the lifetime of the process.

How do you look up information C library calls and system calls?
Use the man pages. Note the man pages are organized into sections. Section 2 = System calls. System 3 = C libraries.
Web: Google "man7 open"
shell: man -S2 open  or man -S3 printf
## How do you allocate memory on the heap?
Use malloc. There's also realloc and calloc.
Typically used with cast and a sizeof. e.g. enough space to hold 10 integers
```
int* space = (int*) malloc(sizeof(int) * 10);
```

## What's wrong with this string copy code?

```C
void mystrcpy(char*dest, char* src) { 
  // void means no return value   
  while( *src ) { dest = src; src ++; dest++; }  
}
```
In the above code it simply changes the dest pointer to point to source string. Also the nul bytes is not copied. Here's a better version - 
```
  while( *src ) { *dest = *src; src ++; dest++; } 
  *dest = *src;
```
Note it's also usual to see the following kind of implementation, which does everything inside the expression test, including copying the nul byte.
```C
  while( (*dest++ = *src++ )) ;


## How do you write a strcpy replacement?
// Use strlen+1 to find the nul byte...
char* mystrdup(char*source) {
   char* p = (char*) malloc ( strlen(source)+1 );
   strcpy(p,source);
   return p;
}
```

How do you unallocate memory on the heap?
Use free!

What is double free? How can you avoid?What is a dangling pointer? How do you avoid?
free(p);
free(p); // Oops!

Fix:
p = NULL;

## What is an example of buffer overflow?
Famous example: Heart Bleed (performed a memcpy into a buffer that was of insufficient size).
Simple example: implement a strcpy and forget to add one to strlen, when determining the size of the memory required.

## What is 'typedef' and how do you use it? 
Declares an alias for a types. Often used with structs to reduce the visual clutter of having to write 'struct' as part of the type.

typedef float real; // abstract the actual type used. In the future we could change this typed and recompile with doubles.
typedef struct link link_t;  //With structs, include the keyword 'struct' as part of the original types


