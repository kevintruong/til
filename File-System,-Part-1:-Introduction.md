# This page is under construction!

## Design a file system! What are your design goals?


## What is  `"."` 	`".."` `"..."` ?
"." represents the current directory
".." represents the previous directory

## What are absolute and relative paths?
Absolute paths are paths that start from the 'root node' of your directory tree. Relative paths are paths that start from your current position in the tree.

## What are some examples of relative and absolute paths?
If you start in your home directory ("~" for short), then "Desktop/cs241" would be a relative path. Its absolute path counterpart might be something like "/Users/[yourname]/Desktop/cs241".

## How do I simplify a/b/../c/./
Remember that ".." means 'parent folder' and that "." means 'current folder'.
Example: a/b/../c/./
- Step 1: cd a (in a)
- Step 2: cd b (in a/b)
- Step 3: cd .. (in a, because .. represents 'parent folder')
- Step 4: cd c (in a/c)
- Step 5: cd . (in a/c, because . represents 'current folder')

Thus, this path can be simplified to a/c

## Why make disk blocks the same size as memory pages?
To support virtual memory, so we can page stuff in and out of memory.

## What information do we want to store for each file?
* Filename
* File size
* Time created, last modified, last accessed
* Permissions
* Filepath
* Checksum
* File data (inode)

## What are the traditional permissions: user – group – other permissions for a file?

## What are the the 3 permission bits for a regular file for each role?
* Read (most significant bit)  
* Write (2nd bit)  
* Execute (least significant bit)

## What do "644" "755" mean?
These are examples of permissions in octal format (base 8). Each octal digit corresponds to a different role (user, group, world).

We can read permissions in octal format as follows:  
* 644 - R/W user permissions, R group permissions, R world permissions  
* 755 - R/W/X user permissions, R/X group permissions, R/X world permissions


## What is an inode? Which of the above items is stored in the inode?



## How does inode store the file contents?
 
Image source: http://en.wikipedia.org/wiki/Ext2 
## How many pointers can you store in each indirection table? 
e.g. With 32 bit addressing and each block is 4KB
Assume 64 bit addressing. Each block is 4KB
