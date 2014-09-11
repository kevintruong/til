## Under constuction!

## What happens when I call malloc?
`malloc` is a C library call and is used to reserve a contiguous block of memory. Unlike stack memory, the memory remains allocated (yours to play with) until free is called with the same pointer. There is also `calloc` and `realloc` which we'll talk about later.

## Can malloc fail?
If malloc fails to reserve any more memory then it returns NULL. Robust programs should check the return value. If your code assumes malloc succeeds and it does not, then your program will likely crash (segfault) when it tries to write to address 0.

## Where is the heap and how big is it? 
A process is a running instance of your program. Each program has it's own address space. For example on a 32 bit machine your process gas about 4 billion addresses to play with, however not all of these are valid or even mapped to actual physical memory (RAM). Inside the process's memory you will find the executable code, space for the stack, environment variables, global (static) variables and the heap.

By calling `sbrk` the C library can increase the size of the heap as your program demands more heap memory. As the heap and stack (one for each thread) need to grow, we put them at opposite ends of the address space. So the heap will grows upwards and the stack grows downwards. If we write a multi-threaded program (more about that later) we will need multiple stacks (one per thread) but there's only ever one heap.

On typical architectures, the heap is part of the `Data segment` and starts just above the code and global variables: 

Do programs need to call brk or sbrk?
Not typically. Instead we call {\tt malloc},{\tt calloc},{\tt realloc} and {\tt free} which are part of the C library. The internal implementation of these functions will call {\tt sbrk} when additional heap memory is required.

What is calloc?
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
Programmers often use `calloc` rather than explicitly calling memset after malloc.
e.g. 
```C
link_t link = calloc(1, sizeof(link_t) ); // Can assume link's memory is zero. 
```

An advanced discussion of these limitations is [[here|http://locklessinc.com/articles/calloc/]]

What is realloc and when would you use it?
`realloc` allows you to resize an existing memory allocation. Using realloc you can request an existing allocation 

How important is that memory allocation is fast?
Very! Allocating and de-allocating heap memory is a common operation in most applications.

What is the silliest malloc and free implementation and what is wrong with it?

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

