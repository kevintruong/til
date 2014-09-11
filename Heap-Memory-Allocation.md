## Under constuction!

## What happens when I call malloc?
`malloc` is a C library call and is used to reserve a contiguous block of memory. Unlike stack memory, the memory remains allocated (yours to play with) until free is called with the same pointer. There is also `calloc` and `realloc` which we'll talk about later.

## Can malloc fail?
If malloc fails to reserve any more memory then it returns NULL. Robust programs should check the return value. If your code assumes malloc succeeds and it does not, then your program will likely crash (segfault) when it tries to write to address 0.

## Where is the heap and how big is it? 
The heap is part of the process memory and it has not a fixed size. Heap memory allocation is performed by the C library when you call malloc (calloc, realloc) and free.

First a quick review on process memory: A process is a running instance of your program. Each program has it's own address space. For example on a 32 bit machine your process gas about 4 billion addresses to play with, however not all of these are valid or even mapped to actual physical memory (RAM). Inside the process's memory you will find the executable code, space for the stack, environment variables, global (static) variables and the heap.

By calling `sbrk` the C library can increase the size of the heap as your program demands more heap memory. As the heap and stack (one for each thread) need to grow, we put them at opposite ends of the address space. So for typical architectures the heap will grows upwards and the stack grows downwards. If we write a multi-threaded program (more about that later) we will need multiple stacks (one per thread) but there's only ever one heap.

On typical architectures, the heap is part of the `Data segment` and starts just above the code and global variables. 

## Do programs need to call brk or sbrk?
Not typically (though calling `sbrk(0)` can be interesting because it tells you where your heap currently ends). Instead programs use `malloc,calloc,realloc` and `free` which are part of the C library. The internal implementation of these functions will call `sbrk` when additional heap memory is required.

## What is calloc?
Unlike `malloc`, `calloc` initializes memory contents to zero and also takes two arguments (the number of items and the size in bytes of each item). A naive but readable implementation of calloc looks like this-
```C
void *calloc(size_t n, size_t size)
{
	size_t total = n * size; // Does not check for overflow!
	void *result = malloc(total);
	
	if (!result) return NULL;
	
// If we're using new memory pages just allocated from the system with sbrk
// then they will be zero so zero-ing out is unnecessary,

	memset(result, 0, total);
	return result; 
}
```
An advanced discussion of these limitations is [[here|http://locklessinc.com/articles/calloc/]]

Programmers often use `calloc` rather than explicitly calling memset after malloc, to set the memory contents to zero. Note `calloc(x,y)` is identical to `calloc(y,x)`

```C
// Ensure our memory is initialized to zero
link_t* link  = malloc(256);
memset(link, 0, 256); // Assumes malloc returned a valid address!

link_t link = calloc(1, 256);
```

```C
## Why doesn't malloc initial memory to zero?
Performance.

## What is realloc and when would you use it?
`realloc` allows you to resize an existing memory allocation. Using realloc you can request an existing allocation 
(todo - example)

## How important is that memory allocation is fast?
Very! Allocating and de-allocating heap memory is a common operation in most applications.

## What is the silliest malloc and free implementation and what is wrong with it?

```C
void* malloc(size_t size)
// Ask the system for more bytes by extending the heap space. sbrk Returns -1 on failure
   void *p = sbrk(size); 
   if(p == (void*) -1) return NULL; // No space left
   return p;
}
void free() {/* Do nothing */}
```
The above implementation suffers from two major drawbacks:
* System calls are slow (compared to library calls). We should reserve a large amount of memory and only occasionally ask for more from the system.
* No reuse of freed memory. Our program never re-uses heap memory - it just keeps asking for a bigger heap.

If this allocator was used in a typical program, the process would quickly exhaust all available memory.
Instead we need an allocator that can efficiently use heap space and only ask for more memory when necessary.

## What are placement strategies?
During program execution memory is allocated ande de-allocated (freed), so there will be gaps (holes) in the heap memory that can be re-used for future memory requests. The memory allocator needs to keep track of which parts of the heap are currently allocated and which are parts are available.

Suppose our current heap size is 64K, though not all of it is in use because some earlier malloc'd memory has already been freed by the program- 

16KB free | 10KB allocated | 1KB free | 1KB allocated | 30KB free | 4KB allocated | 2KB free 
---|---|---|---|---|---|---

If a new malloc request for 2KB is executed (`malloc(2048)`), where should `malloc` reserve the memory? It could use the last 2KB hole (which happens to be the perfect size!) or it could split one of the other two free holes. These choices represent different placement strategies.


A perfect-fit strategy finds the smallest hole that is of sufficient size (at least 2KB):

16KB free | 10KB allocated | 1KB free | 1KB allocated | 30KB free | 4KB allocated | 2KB HERE!
---|---|---|---|---|---|---|---

A worst-fit strategy finds the largest hole that is of sufficient size (so break the 30KB hole into two):

16KB free | 10KB allocated | 1KB free | 1KB allocated | 2KB HERE!| 28KB free | 4KB allocated | 2KB free 
---|---|---|---|---|---|---|---

A first-fit strategy finds the first available hole that is of sufficient size (break the 16KB hole into two):

2KB HERE! | 14KB free | 10KB allocated | 1KB free | 1KB allocated | 30KB free | 4KB allocated | 2KB free 
---|---|---|---|---|---|---|---


## What are the challenges of writing a good allocator?
