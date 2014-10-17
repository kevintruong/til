What is a pipe?

A POSIX pipe is almost like its real counterpart - you can stuff bytes down one end and they will appear at the other end in the same order. Unlike real pipes however, the flow is always in the same direction, one file descriptor is used for reading and the other for writing. The `pipe` system call is used to create a pipe.
```C
int filedes[2];
pipe( filedes );
printf("read from %d, write to %d\n", filedes[0], filedes[1]);
```

These file descriptors can be used with `read` -
```C
// To read...
char buffer[80];
int bytesread = read(filedes[0], buffer, sizeof(buffer));
```
And `write` - 
```C
write(filedes[1], "Go!", sizeof("Go!"));
```

## How can I use pipe to communicate with a child process?
A common method of using pipes is to create the pipe before forking.
```
int filedes[2];
pipe( filedes );
pid_t child = fork();
if(child > 0) { /* I must be the parent */
    char buffer[80];
    int bytesread = read(filedes[0] , buffer, sizeof(buffer));
    // do something with the bytes read    
}
```

The child can then send a message back to the parent:
```
if(child ==0) {
   write(filedes[1], "done", 4);
}
```

## Pipe Gotchas (1)
Here's a complete example that doesn't work! The child reads one byte at a time from the pipe and prints it out - but we never see the message! Can you see why?

```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

int main() {
    int fd[2];
    pipe(fd);
    //You must read from fd[0] and write from fd[1]
    printf("Reading from %d, writing to %d\n",fd[0], fd[1]);

    pid_t p = fork();
    if(p>0) {
        /* I have a child therefore I am the parent*/
        write(fd[1],"Hi Child!",9);
    }else {
        char buf;
        int bytesread;
        // read one byte at a time.
        while((bytesread=read(fd[0],&buf,1)) > 0) {
            putchar(buf);
        }
    }
    return 0;
}

The parent sends the bytes `H,i,(space),C...!` into the byte (this may block if the pipe is full).
The child starts reading the pipe one byte at a time. In the above case, the child process will read and print each character. However it never leaves the while loop! When there are no characters left to read it simply blocks and waits for more. The call `putchar` writes the characters out but we never flush the buffer.

To see the message we could flush the buffer (e.g. fflush(stdout) or printf("\n"))
or better, let's look for the end of message '!'
```C
        while((bytesread=read(fd[0],&buf,1)) > 0) {
            putchar(buf);
            if(buf == '!') break; /* End of message */
        }
```
And the message will be flushed to the terminal when the child process exits.


## Using pipes with fdopen




## When do I need two pipes?

If you need to send data to and from a child, then two pipes are required (one for each direction).
Otherwise the child would attempt to read its own data intended for the parent (and vice versa)

## Closing pipes gotchas

Processes receive the signal SIGPIPE when no one is listening! From the pipe(2) man page - 
```
If all file descriptors referring to the read end of a pipe have been closed,
 then a write(2) will cause a SIGPIPE signal to be generated for the calling process. 
```
Tip: Notice only the writer (not a reader) can use this signal.
To inform the reader that a writer is closing their end of the pipe, you could write your own special byte (e.g. 0xff) or a message ( `"Bye!"`)

Here's an example of catching this signal that doesnt work!

Todo

## The lifetime of pipes
Unnamed pipes live in memory (do not take up any disk space) and is a simple and efficient form of interprocess communication that is useful for streaming data. Once all processes have closed, the pipe resources are freed.

An alternative to _unamed_ pipes is _named_ pipes created using `mkfifo` - more about these in a future lecture.

