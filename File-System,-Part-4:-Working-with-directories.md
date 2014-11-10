## How do I find out if file (an inode) is a regular file or directory?

Use the `S_ISDIR` macro to check the mode bits in the stat struct:

```C
struct stat s;
stat("/tmp", &s );
if( S_ISDIR(s.st_mode) ) { ... 
```
Note, more robust code would verify that the stat call succeeds (returns 0).

## How do I recurse into subdirectories?

First a puzzle - how many bugs can you find in the following code?
```C
void dirlist(char*path) {
  
  struct dirent* dp;
  DIR* dirp = opendir(path);
  while ((dp = readdir(dirp)) != NULL) {
     char newpath[strlen(path) + strlen(dp->d_name) + 1];
     sprintf(newpath,"%s/%s", newpath, dp->d_name);
     printf("%s\n", dp->d_name);
     dirlist(newpath);
  }
}

int main(int argc, char**argv) { dirlist(argv[1]);return 0; }
```
Did you find all 5 bugs?
```C
// Check opendir result (perhaps user gave us a path that can not be opened as a directory
if(!dirp) { perror("Could not open directory"); return }
// +2 as we need space for the / and the terminating 0
char newpath[strlen(path) + strlen(dp->d_name) + 2]; 
// Correct parameter
sprintf(newpath,"%s/%s", path, dp->d_name); 
// Perform stat test (and verify) before recursing
if( 0==stat(newpath,&s) && S_ISDIR(s.st_mode)) dirlist(newpath)
```


## What are symbolic links? How do they work? How do I make one?
```C
symlink(const char *target, const char *symlink);
```
To create a symbolic link in the shell use `ln -s`

To read the contents of the link as just a file use `readlink`
```
> readlink myfile.txt
../../dir1/notes.txt
```

To read the meta-(stat) information of a symbolic link use `lstat` not `stat`
```
struct stat s;
stat("myfile.txt", &s1); // stat info about  the notes.txt file
lstat("myfile.txt", &s2); // stat info about the symbolic link
```


## Advantages of symbolic links
Can refer to a files that don't exist yet
Unlike hard links, can refer to directories.
Can refer to files (and directories) that exist outside of the current file system

## What is `/dev/null` and when is it used?

The file `/dev/null` is a great place to store bits that you never need to read!
Bytes sent to `/dev/null/` are never stored - they are simply discarded. A common use of `/dev/null` is to discard standard output. For example,
```
ls . >/dev/null
```

## Why would I want to set a directory's sticky bit?

When a directory's sticky bit is set only the file's owner, the directory's owner, and the root user can rename (or delete) the file. This is useful when multiple users have write access to a common directory

A common use of the sticky bit is for the shared and writable `/tmp` directory.


## Why do shell and script programs start with `#!/usr/bin/env python` ?



## How do I make 'hidden' files i.e. not listed by "ls"? How do I list them?
Easy! Create files (or directories) that start with a "." - then (by default) they are not displayed by standard tools and utilities.

This is often used to hide configuration files inside the user's home directory.
For example `ssh` stores its preferences inside a directory called `.sshd`

To list all files including the normally hidden entries use `ls` with  `-a` option 
```
>ls -a
.			a.c			myls
..			a.out			other.txt
.secret	
```



## What happens if I turn off the execute bit on directories?
The execute bit for a directory is used to control whether the directory contents is listable.

```
>chmod ugo-x dir1
>ls -l
drw-r--r--   3 angrave  staff   102 Nov 10 11:22 dir1
```

However when attempting to list the contents of the directory,
```
>ls dir1
ls: dir1: Permission denied
```
In other words, the directory itself is discoverable but its contents cannot be listed.



## What is file globbing (and who does it)?
Todo


## What is the security  problem with the following?
```C
mkdir /tmp/mystuff
chmod 700 /tmp/mystuff
```
Todo

## My default umask 022; what does this mean?
Todo



