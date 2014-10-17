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

## Using pipes with fdopen

Todo : `fdopen` example

## When do I need two pipes?

If you need to send data to and from a child, then two pipes are required (one for each direction).
Otherwise the child would attempt to read its own data intended for the parent (and vice versa)

## Closing pipes gotchas
Todo
## The lifetime of pipes
Todo