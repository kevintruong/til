# This page is under construction!

## Design a file system! What are your design goals?


## What is  `"."` 	`".."` `"..."` ?
"." represents the current directory
".." represents the previous directory

## What are absolute and relative paths?
Absolute paths are paths that start from the 'root node' of your directory tree. Relative paths are those that start from your current position in the tree.

## What is an example of a relative path? What about an absolute path?
If you start in your home directory ("~" for short), then "Desktop/cs241" would be a relative path. Its absolute path counterpart might be something like "/Users/[yourname]/Desktop/cs241".

## How do I simplify a/b/../c/./

## Why make disk blocks the same size as memory pages?


## What information do we want to store for each file?

## What are the traditional permissions: user – group – other permissions for a file?

## What are the the 3 permission bits for a regular file for each role?

## What do "644" "755" mean?



## What is an inode? Which of the above items is stored in the inode?



## How does inode store the file contents?
 
Image source: http://en.wikipedia.org/wiki/Ext2 
## How many pointers can you store in each indirection table? 
e.g. With 32 bit addressing and each block is 4KB
Assume 64 bit addressing. Each block is 4KB
