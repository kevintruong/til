
A memory allocator needs to keep track of which bytes are currently allocated and which are which are available for use. This page introduces some of the implementation and conceptual details of building an allocator i.e. the actual code that implements malloc and free.

## This page talks about links of blocks - do I malloc memory for them instead?
Though conceptually we are thinking about creating linked lists and lists of blocks, we don't need to "malloc memory" to create them! Instead we are writing integers and pointers into memory that we already control so that we can later consistent hop from one address to the next. This internal information represents some overhead. So even if we had requested 1024 KB of contiguous memory from the system, we will not be able to provide all of it to the running program.

## Thinking in blocks
We can think of our heap memory as a list of blocks where each block is either allocated or unallocated.
Rather than storing an explicit list of pointers we store information about the block's size _as part of the block_. Thus there is a conceptually a list of free blocks, but it is implicit; i.e. in the form of block size information that we store as part of each block 

We could navigate from one block to the next block just by adding the block's size. For example if you have a pointer 'p' that points to the start of a block, then `next_block`  with be at `((char*)p) +  *(size_t*) p`, if you are storing the size of the blocks in bytes. The cast to char* ensures that pointer arithmetic is calculated in bytes. The cast to `size_t*` ensures the memory at p is read as a size value and would be necessarily if p was a `void*` or `char*` type.

The calling program never sees these values; they are internal to the implementation of the memory allocator. 

As an example, suppose your allocator is asked to reserve 80 bytes (`malloc(80)`) and requires 8 bytes of internal header data. The allocator would need to find an unallocated space of at least 88 bytes. After updating the heap data it would return a pointer to the block. However the returned pointer does not point to the start of the block because that's where the internal size data is stored! Instead we would return the start of the block + 8 bytes.
In the implementation, remember that pointer arithmetic depends on type. For example, `p += 8` adds `sizeof(p)*8` not necessarily 8 bytes!

## Implementing malloc
The simplest implementation uses first fit. Start at the first block,if it exists, and iterate until a block that represents unallocated space of at least the required size is found.

If no block is found then it's time to call `sbrk()` again to extend the size of the heap. A fast implementation might extend it a significant amount so that we will not need to request more heap memory in the near future.

When a free block is found it may be larger than the space we need. If so, we will create two entries in our implicit list. The first entry is the allocated block, the second entry is the remaining space.

There are two simple ways to store if a block is in use or available. Either store it as a byte in the header information along with the tail. An alternative is to encode it as the lowest bit in the size!
Thus size information would be limited to only even values:
```
isallocated = (*p) & 1;
realsize = (*p) & ~1;  // mask out the lowest bit
```

## Implementing free
When `free` is called we need to re-apply the offset to get back to the 'real' start of the block i.e. to where we stored the size information.

A naive implementation would simply mark the block as unused. If we storing the block allocation status in the lowest size bit, then we just need to clear the bit:
```C
*p = *p & ~1; // Clear lowest bit 
```
However we have a bit more work to do: If the current block and the next block, if it exists, are both free we need to coalesce these blocks into a single block.
Similarly we also need to check the previous block too. If that exists and represents an unallocated memory, then we need to coalesce the blocks into a single large block.

To be able to coalesce a free block with a previous free block we will also need to find the previous block, so we store the block's size at the end of the block too. These are called "boundary tags" (ref Knuth73). As the blocks are contiguous, the end of one blocks sits right next to the start of the next block. So the current block (apart from the first one) can look a few bytes further back to lookup the size of the previous block. With this information you can now jump backwards!

(Hey world - an open source licensed diagram of the above would be nice)

## Performance
With the above description it's possible to build a memory allocator. It's main advantage is simplicity - at least simple compared to other allocators.
Allocating memory is a worst-case linear time operation (search linked lists for a sufficiently large free block) and De-allocation is constant time (no more than 3 blocks will need to be coalesced into a single block). Using this allocator it is possible to experiment with different placement strategies.

## Explicit Free Lists Allocators

Better performance can be achieved by implementing an explicit doubly-linked-list of free nodes i.e. we can immediately traverse to the next free block and the previous free block. This can halve the search time because the linked list only includes unallocated blocks.

A second advantage is that we now have some control over the ordering the linked list. For example when a block is freed, we could choose to insert it into the beginning of the linked-list, rather than always between its neighbors. This is discussed below

Where do we store the pointers of our linked list? A simple trick is realize that the block itself is not being used, so we can store the next and previous pointers as part of the block.

We still need to implement Boundary Tags (i.e. an implicit list using sizes), so that we can correctly free blocks and coalesce them with their two neighbors. Thus explicit free lists require more code and complexity.

With explicit linked lists a fast and simple 'find-first' algorithm is used to find the first sufficiently large link. However since the link order can be modified, this corresponds to different placement strategies. For example if the links are maintained in largest to smallest, then this produces a Worst-fit placement strategy.

### Explicit Linked list insertion policy
The newly freed block can be inserted easily into two possible positions: at the beginning and or in address order (by using the boundary tags to first find the neighbors).

Inserting at the beginning creates a LIFO (last-in-first-out) policy: The most recently freed spaces will be re-used. Studies suggest fragmentation is worse than address ordered.

Inserting in address order  ("Address ordered policy") inserts freed blocks so that the blocks are visited in increasing address order. This policy required more time to free a block because the boundary tags (size data) must be used to find the next and previous unallocated blocks however there is less fragmentation.

# Case study: Buddy Allocator (an example of a segregated list)
* See [[http://books.google.com/books?id=0uHME7EfjQEC&printsec=frontcover#v=onepage&q&f=false]] (goto page 85 or search for 'buddy')

See the powerpoint 

* [[https://subversion.ews.illinois.edu/svn/fa14-cs241/_shared/lectures/CS241-05-ThanksForTheMemorySlides.pptx]] (powerpoint)
* [[https://subversion.ews.illinois.edu/svn/fa14-cs241/_shared/lectures/CS241-05-ThanksForTheMemorySlides.pptx.pdf]] (pdf)
and 
* [[http://en.wikipedia.org/wiki/Buddy_memory_allocation]]
# Other allocators
There are many other allocation schemes. For example [[SLUB|http://en.wikipedia.org/wiki/SLUB_%28software%29]] (wikipedia) - one of three allocators used internally by the Linux Kernel.