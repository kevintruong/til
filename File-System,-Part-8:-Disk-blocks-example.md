## Under construction

## Please can you explain a simple model of how the file's content is stored in a simple i-node based filesystem?

Sure! To answer this question we'll build a virtual disk and then write some C code to access its contents. Our filesystem will divide the bytes available into space for inodes and a much larger space for disk blocks. Each disk block will be 4096 bytes- 

```C
#define MAX_INODE (1024)
#define MAX_BLOCK (1024*1024)

typedef char[4096] block_t;

struct inode[MAX_INODE] inodes;

block[MAX_BLOCK] blocks;
```

Note for clarity we will not use 'unsigned' in this example. Our simplest inode will contain the file's size in bytes, permission,user,group information, time meta data and ten pointers to disk blocks. 

```C
struct inode {
 int[10] directblocks;
 long size;
 int mode, userid,groupid;
 int ctime,atime,mtime;
}
```
Now we can read a byte from one of those ten blocks.
```C
char readbyte(inode*inode,long position) {
  if(position <0 || position >= inode->size) return -1;

  int  block_count = position / 4096,offset = position % 4096;
  
  // block count better be 0..9 !
  int physical_idx = lookup_physical_block_index(inode, block_count );

  // read the  disk block from our virtual disk and return the specific byte
  return blocks[physical][offset];
}
```
Our initial version of lookup_physical_block is simple - just use our table of 10 direct blocks!

```C
int lookup_physical_block_index(inode*inode, int block_count) {
  assert(block_count>=0 && block_count < 10);
  return inode->directblocks[ block_count ];
}
```


This simple representation is reasonable provided we can represent all possible files with just ten blocks. What about larger files? We need the inode struct to always be the same size so just increasing the direct block array to 20 will roughly doudly the size of our inodes. If most of our files require less than 10 blocks, then our inode array is now wasteful. To solve this problem we will use a disk block to extend the array of pointers at our disposal. We will only need this for files > 40KB

```C
struct inode {
 int[10] directblocks;
 int indirectblock;
 ...
}
```

The indirect block is just a regular disk block of 4096 bytes but we will use it to hold pointers to disk blocks. Our pointers in this case are just integers, so we need to cast the pointer to an integer pointer:


```C
int lookup_physical_block_index(inode*inode, int block_count) {
  assert(block_count>=0 && block_count < 1024 + 10);
  if( block_count < 10)
     return inode->directblocks[ block_count ];
  
  // read the indirect block ...
  block_t* oneblock = & blocks[ inode->indirectblock ];

  // Treat it as an array of pointers to disk blocks
  int* table = (int*) oneblock;
  
  // Look up the correct entry (offset by 10 because the first 10 blocks of the file are already 
 // accounted for
  return table[ block_count - 10 ];
}
```


For a typical filesystem our index values are 32 bits i.e. 4bytes. Thus in 4096 bytes we can store 4096 / 4 = 1024 entries
This means our indirect block can refer to 1024 * 4KB = 4MB of data. With the first ten direct blocks we can there accommodate files up to 4136KB (4096+40). Entries which correspond to files larger than the actual file size are considered to be invalid and never used.

For files larger than ~4MB, we could use two indirect blocks. However there's a better alternative, that will allow us to efficiently scale up to huge files. We will include a double-indirect pointer and if that's not enough a triple indirect pointer.

```C
int lookup_physical_block_index(inode*inode, int block_count) {
  if( block_count < 10)
     return inode->directblocks[ block_count ];

  // Use indirect block as an extended table:
  // Assumes 1024 ints can fit inside each block!
  if( block_count < 1024 + 10) {   
      int* table = (int*) & blocks[ inode->indirectblock ];
      return table[ block_count - 10 ];
  }
  // For huge files we will use a table of tables
  int i = (block_count - 1034) / 1024 , j = (block_count - 1034) % 1024;
  assert(i<1024);

  int* table1 = (int*) & blocks[ inode->doubleindirectblock ];
   // The first table tells us where to read the second table ...
  int* table2 = (int*) & blocks[   table1[i]   ];
  return table2[j];
}
```
Todo: Include a picture



