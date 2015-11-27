> Question numbers subject to change

## Q1
Use POSIX `pipe` `dup2` and `close` to implement an autograding program. Capture the standardoutput of a child process into a pipe. The child process should `exec` the program `./test` with no additional arguments (other than the process name). In the parent process read from the pipe: Exit the parent process as soon as the captured output contains the ! character. Before exiting the parent process send SIGKILL to the child process. Exit 0 if the output contained a !. Otherwise if the child process exits and closes the pipe then exit with a value of 1.

