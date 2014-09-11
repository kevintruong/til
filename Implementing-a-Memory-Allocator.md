A memory allocator needs to keep track of which bytes are currently allocated and which are which are available for use. This page discusses some of the implementation and conceptual details of building an allocator i.e. the actual code that implements malloc and free.

Sorry - this page is still under construction. 
For now, please refer to  [[https://subversion.ews.illinois.edu/svn/fa14-cs241/_shared/lectures]]

FYI: Though conceptually we are thinking about creating links in a linked lists, we don't need to "malloc memory" to create the linked lists! Instead we are writing integers and pointers into memory that we control so that we can later consistent hop from one address to the next.

## 
We can think of our heap memory as a list of blocks where each block is either allocated or unallocated.
Rather than storing an explicit list of pointers we store information about the block's size as part of the block. Thus there is a list of free blocks, but it is in the form of block size information that we store as part of each block.

We could navigate from one block to the next block just by adding the block's size. For example if you have a pointer 'p' that points to the start of a block, then `next_block`  with be at `((char*)p) +  *(size_t*) p`, if you are storing the size of the blocks in bytes. The cast to char* ensures that pointer arithmetic is calculated in bytes. The cast to `size_t*` ensures the memory at p is read as a size value and would be necessarily if p was a `void*` or `char*` type.

The calling program never sees these values; they are internal to the implementation of the memory allocator. 

As an example, suppose your allocator is asked to reserve 80 bytes (`malloc(80)`) and requires 8 bytes of internal header data. The allocator would need to find an unallocated space of at least 88 bytes. After updating the heap data it would return a pointer to the block. However the returned pointer does not point to the start of the block because that's where the internal size data is stored! Instead we would return the start of the block + 8 bytes.
In the implementation, remember that pointer arithmetic depends on type. For example, `p += 8` adds `sizeof(p)*8` not necessarily 8 bytes!

### Implementing malloc
The simplest implementation uses first fit. Start at the first block,if it exists, and iterate until a block that represents unallocated space of at least the required size is found.

If no block is found then it's time to call `sbrk()` again to extend the size of the heap. A fast implementation might extend it a significant amount so that we will not need to request more heap memory in the near future.

When a free block is found it may be larger than the space we need. If so, we will create two entries in our implicit list. The first entry is the allocated block, the second entry is the remaining space.

There are two simple ways to store if a block is in use or available. Either store it as a byte in the header information along with the tail. An alternative is to encode it as the lowest bit in the size!
Thus size information would be limited to only even values:

isallocated = (*p) & 1;
realsize = (*p) & ~1;  // mask out the lowest bit


### Implementing free
When `free` is called we need to re-apply the offset to get back to the 'real' start of the block i.e. to where we stored the size information.

A naive implementation would simply mark the block as unused. If we storing the block allocation status in the lowest size bit, then we just need to clear the bit:
```C
*p = *p & ~1; // Clear lowest bit 
```
However we have a bit more work to do: If the current block and the next block, if it exists, are both free we need to coalesce these blocks into a single block.
Similarly we also need to check the previous block too. If that exists and represents an unallocated memory, then we need to coalesce the blocks into a single large block.

To be able to coalesce a free block with a previous free block we will also need to find the previous block, so we store the block's size at the end of the block too (ref Knuth73). As the blocks are contiguous, the end of one blocks sits right next to the start of the next block. So the current block (apart from the first one) can look a few bytes further back to lookup the size of the previous block. With this information you can now jump backwards!

(Hey world - an open source licensed diagram of above would be nice)

## Explicit Free Lists Allocators



# Case study: Buddy Allocator

# Other allocators
There are many other allocation schemes for example [[SLUB|http://en.wikipedia.org/wiki/SLUB_%28software%29]] (wikipedia) - one of three allocators used internally by the Linux Kernel.