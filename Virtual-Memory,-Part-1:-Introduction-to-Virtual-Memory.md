## What is Virtual Memory?

In very simple embedded systems and early computers, processes directly access memory i.e. "Address 1234" corresponds to a particular byte stored in a particular part of physical memory.
In modern systems, this is no longer the case. Instead each process is isolated; and there is a translation process between the address of a particular CPU instruction or piece of data of a process and the actual byte of physical memory ("RAM"). Memory addresses are no longer 'real'; the process runs inside virtual memory. Virtual memory non only keeps processes safe (because one process cannot directly read or modify another process's memory) it also allows the system to efficiently allocate and re-allocate portions of memory to different processes.

## What is the MMU?
The Memory Management Unit is part of the CPU. It converts a virtual memory address into a physical address. The MMU may also interrupt the CPU if there is currently no mapping from a particular virtual address to a physical address or if the current CPU instruction attempts to write to location that the process only has read-access.

## So how do we convert a virtual address into a physical address.
Imagine you had a 32 bit machine. Pointers can hold 32 bits i.e. they can address 2^32 different locations i.e. 4GB of memory (we will following the standard convertion of one address can hold one byte).

Imagine we had a large table - here's the clever part - stored in memory! For every possible address (all 4 billion of them) we will store the 'real' i.e. physical address. Each physical address will need 4 bytes (to hold the 32 bits).
This scheme would require 16 billion bytes to store all of entries. Oops - our lookup scheme would consume all of the memory that we could possibly buy for our 4GB machine.
We need to do better than this. Our lookup table better be smaller than the memory we have otherwise we will have no space left for our actual programs and operating system data.
The solution is to chunk memory into small regions called 'pages' and 'frames' and use a lookup table for each page.
# What is a page? How many of them are there?

A page is block of virtual memory. A typical block size on Linux operating system is 4KB (i.e. 2^12 addresses), though you can find examples of larger blocks.

So rather than talking about individual bytes we can talk about blocks of 4KBs. Each block is called a page. We can also number our pages ("Page 0" "Page 1" etc)

## EX: How many pages are there in a 32bit machine (assume page size of 4KB)?
Answer: 2^32 address / 2^12 = 2^20 pages.

Remember that 2^10 is 1024, so 2^20 is a bit more than one million.

For a 64 bit machine, 2^64 / 2^12 = 2^52, which is roughly 10^5 pages.
## What is frame?
A frame (or sometimes called a 'page frame') is block of physical memory or RAM (=Random Access Memory)
A frame is the same size of virtual page; so there will be the same number of them in the addressable space of the machine.

## What is a page table and how big is it?
A page table is a mapping between a page to the frame.
For example Page 1 might be mapped to frame 45, page 2 mapped to frame 30. Other frames might be currently unused or assigned to other running processes, or used internally by the operating system.

A simple page table is just an array, `int frame = table[ page_num ];`

For a 32 bit machine with 4KB pages, each entry needs to hold a frame number - i.e. 20 bits because we calculated there are 2^20 frames. That's 2.5 bytes per entry! In practice, we'll round that up to 4 bytes per entry and find a use for those spare bits. With 4 bytes per entry x 2^20 entries = 4 MB of physical memory are required to hold the page table.

For a 64 bit machine with 4KB pages, each entry needs 52 bits. Let's round up to 64 bits (8 bytes) per entry. With 2^52 entries thats 2^55 bytes (roughly 40 peta bytes...) Oops our page table is too large.

In 64 bit architectures memory addresses are sparse, so we need a mechanism to reduce the page table size, given that most of the entries will never be used.

## Multi-level page tables
