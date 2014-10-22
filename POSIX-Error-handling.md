## What is `errno` and when is it set?
	
POSIX defines a special integer `errno` that is set when a system call fails.
The initial value of `errno` is zero (i.e. no error).
When a system call fails it will typically return -1 to indicate an error and set `errno`

## What about multiple threads?
Each thread has it's own copy of `errno`. This is very useful; otherwise an error in one thread would interfere with the error status of another thread.

## When is errno reset to zero?
It's not unless you specifically reset it to zero!  When system calls are successful they do _not_ reset the value of `errno`.

This means you should only rely on the value of errno if you know a system call has failed (e.g. it returned -1).

## What are the gotchas and best practices of using errno?
Be careful when complex error handling use of library calls or system calls that may change the value of `errno`. In practice it's safer to copy the value of errno into a int variable:
```C
// Unsafe - the first fprintf may change the value of errno!!
if( -1 == sem_wait(&s)) {
   fprintf(stderr,"An error occurred!");
   fprintf(stderr,"The error value is %d\n",errno);
}
// Better, copy the value before making more system and library calls
if( -1 == sem_wait(&s)) {
   int errno_saved = errno;
   fprintf(stderr,"An error occurred!");
   fprintf(stderr,"The error value is %d\n",errno_saved);
}
```

In a similar vein, if your signal handler makes any system or library calls, then it is good practice to save the original value of errno and restore the value before returning:

```C
void handler(int signal) {
   int errno_saved = errno;
   // do stuff that might change errno

   errno = errno_saved;
}
```
## How can you print out the string message associated with a particular error number?

Use `strerror` to get a short (English) description of the error value.

```C
char* mesg = strerror(errno);
fprintf(stderr, "An error occurred (errno=%d): %s",errno, mesg);
```

## What are the gotchas of using strerror?
strerror is not threadsafe. In other words, two threads cannot call it at the same time!

There are two workarounds: Firstly we can use a mutex lock to define a critical section and a local buffer. The same mutex should be used by all threads in all places that call `strerror`
```C
pthread_mutex_lock(&m);
char* result = strerror(errno);
char* message = malloc(strlen(mesg+1));
strcpy(message, result);
pthread_mutex_unlock(&m);
fprintf(stderr, "An error occurred (errno=%d): %s",errno, message);
free(message);
```

Alternatively use the less portable but thread-safe `strerror_r`

## What is EINTR? What does it mean for sem_wait? read? write?

Some system calls can be interrupted when a signal (e.g SIGCHLD, SIGPIPE,...) is delivered to the process. At this point the system call may return without performing any action! For example, bytes may not have been read/written, semaphore wait may not have waited.

This interruption can be detected by checking the return value and if `errno` is EINTR. In which case the system call should be retried. It's common to see the following kind of loop that wraps a system call (such as sem_wait).

```C
while(( -1 == systemcall(...)) && (errno == EINTR) ) { /* repeat! */}
```