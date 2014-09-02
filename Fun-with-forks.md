## What does fork do?

fork clones the current process to create a new process. It creates a new process (the child process) by duplicating the state of the existing process with a few minor differences (discussed below). The child process does not start from main. Instead it returns from fork() just as the parent process does.


## What is the simplest fork() example?
Here's a very simple example...
```C
printf("I'm printed once!\n");
fork();
// Now there are two processes running, and each process will print out the next line.
printf("You see this line twice!\n");
```

## Why does this example print 42 twice?
The following program prints out 42 twice - but the fork() is after the printf!? Why?
```C
#include <unistd.h>
#include <stdio.h>
int main() {
   int answer = 84 >> 1;
   printf("Answer: %d", answer);
   fork();
   return 0;
}
```
The printf line _is_ executed only once however notice that the printed contents is not flushed to standard out (there's no newline printed, we didn't called `fflush`, or change the buffering mode).
The output text is therefore in still in process memory waiting to be sent.
When fork() is executed the entire process memory is duplicated including the buffer. Thus the child process starts with a non-empty output buffer which will be flushed when the program exits.

## How do you write code that is different for the parent and child process?

Use the return value of fork(). The child process knows its parent - the original process that was duplicated -  by calling getppid() (note the double p) - so does not need any additional return information form fork(). The parent process however can only find out the id of the child process from the return value of fork:
```C
int id = fork();
if( id== -1) exit(1); // fork failed 
if( id > 0 )
{ 
\\ I'm the original parent and I just created a child process with id 'id'
} else {
\\I must be the newly made child process, that looks almost identical to the parent getppid()
}

