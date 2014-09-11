A memory allocator needs to keep track of which bytes are currently allocated and which are which are available for use. This page discusses some of the implementation and conceptual details of building an allocator i.e. the actual code that implements malloc and free.

Sorry - this page is still under construction. 
For now, please refer to  [[https://subversion.ews.illinois.edu/svn/fa14-cs241/_shared/lectures]]

FYI: Though conceptually we are thinking about creating links in a linked lists, we don't need to "malloc memory" to create the linked lists! Instead we are writing integers and pointers into memory that we control so that we can later consistent hop from one address to the next.

## Implicit Free Lists
We can think of our heap memory as a list of blocks where each block is either allocated or unallocated.
Rather than storing an explicit list of pointers we store information about the block's size as part of the block.

Thus we could navigate from one block to the next block just by adding the block's size. For example if you have a pointer 'p' of right type (for simplicity imagine an unsigned integer pointer) that points to the start of a block, then `next_block = p + *p`;

The calling program never sees these values; they are internal to the implementation of the memory allocator. 

As an example, suppose your allocator is asked to reserve 80 bytes (`malloc(80)`) and requires 8 bytes of internal header data. The allocator would need to find an unallocated space of at least 88 bytes. After updating the heap data it would return a pointer to the block. However the returned pointer does not point to the start of the block because that's where the internal size data is stored! Instead we would return the start of the block + 8 bytes.
In the implementation, remember that pointer arithmetic depends on type. For example, `p += 8` adds `sizeof(p)*8`!

### Implementing free
When `free` is called. We need to re-apply the offset to get back to the real start of the block.
A naive implementation would simple mark the block as unused.
However we have a bit more work to do: If the current block and the next block are both available we need to coalesce these blocks into a single block.

### Details

In practice, to be able to coalesce a free block with a previous free block we will also need to find the previous block, so we store the block's size at the end of the block too (ref Knuth73). As the blocks are contiguous, the end of one blocks sits right next to the start of the next block. So the current block (apart from the first one) can look a few bytes further back to lookup the size of the previous block. With this information you can now jump backwards!

(Hey world - an open source licensed diagram of above would be nice)








### Coalescing blocks

The allocator needs to join 


## Explicit Free Lists Allocators



Case study: Buddy Allocator