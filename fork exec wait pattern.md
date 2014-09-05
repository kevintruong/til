What does the following 'exec' example do?
```C
int main() {
   close(1); // close standard out
   open("log.txt", O_RDWR | O_CREAT | O_APPEND, S_IRUSR | S_IWUSR);
   puts("Captain's log");
   chdir("/usr/include");
   execl("/bin/ls", "/bin/ls",".",(char*)NULL); // "ls ."
   perror("exec failed");
   return 0; // Not expected
}
```
What does the following program do and how does it work?

## What does the child inherit from the parent?
* Open filehandles. If the parent later seeks, say, to the back to the beginning of the file then this will affect the child too (and vice versa).
* Signal handlers
* Current working directory.

## What is different in the child process than the parent process?
The process id is different. In the child calling `getppid()` (notice the two 'p's) will give the same result as calling getpid() in the parent.

## How do I wait for my child to finish?
Use `waitpid` or `wait`). The parent process will pause until `wait` (or `waitpid`) returns. Note this explanation glosses over the restarting discussion.


## Can I find out the exit value of my child?
You can find the lowest 8 bits of the child's exit value (the return value of `main()` or value included in `exit()`): Use the macros (see `wait`/`waitpid` man page) and the `wait` or `waitpid` call
```C
int status;
pid_t child = fork();
if(fork>0) {
  pid_t pid = waitpid(child, &status, 0);
  if(pid != -1 && WIFEXITED(status) {
     int low8bits = WEXITSTATUS ( status );
     printf("Process %d returned %d" , low8bits);
  }
}
```


## How do I start a background process that runs as the same time.?
Don't wait for them! Your parent process can continue to execute code without having to wait for the child process. Note in practice background processes can also be disconnected from the parent's input and output streams by calling `close` on the open file descriptors before calling exec.

However child processes that finish before their parent finishes can become zombies. See the zombie page.