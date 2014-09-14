## The Hitchhiker's Guide to Debugging C Programs

Feel free to add anything that you found helpful in debugging C programs including but not limited to, debugger usage, recognizing common error types, gotchas, and effective googling tips.

### Should I ask a question on Piazza?
Here are a few things you should try/check for before asking on Piazza - you'll usually get a meaningful response faster this way.
0. Am I running on EWS? (You should be.)
1. Did I check the man pages for the failing command(s)? (Eg. To check for forgotten includes)
2. Are all my includes good?
3. Are there any silly errors near the bug that I can see just by looking at the code? (E.g. a + instead of a - etc.)
4. Did I spend at least 15 minutes Googling the error?
5. Did I try commenting out parts of the code bit by bit to find out precisely where the error occurs?
6. Did I commit my code to SVN in case the TAs need more context?
7. Did I include the console/GDB/Valgrind output + code surrounding the bug in my Piazza post?

### Most Common Errors

@todo: add some common error messages in system programming with examples, causes, and common debugging approaches.
#### Segmentation fault
### GDB

Introduction: http://www.cs.cmu.edu/~gilpin/tutorial/

#### Setting breakpoints programmatically

A very useful trick when debugging complex C programs with GDB is setting breakpoints in the source code.

```c
int main() {
    int val = 1;
    val = 42;
    asm("int $3"); // set a breakpoint here
    val = 7;
}
```

```sh
$ gcc main.c -g -o main && ./main
(gdb) r
[...]
Program received signal SIGTRAP, Trace/breakpoint trap.
main () at main.c:6
6	    val = 7;
(gdb) p val
$1 = 42
```



#### Checking memory content

http://www.delorie.com/gnu/docs/gdb/gdb_56.html

For example,

```c
int main() {
    char bad_string[3] = {'C', 'a', 't'};
    printf("%s", bad_string);
}
```

```sh
$ gcc main.c -g -o main && ./main
$ Cat ZVQ� $
```

```sh
(gdb) l
1	#include <stdio.h>
2	int main() {
3	    char bad_string[3] = {'c', 'a', 't'};
4	    printf("%s", bad_string);
5	}
(gdb) b 4
Breakpoint 1 at 0x100000f57: file main.c, line 4.
(gdb) r
[...]
Breakpoint 1, main () at main.c:4
4	    printf("%s", bad_string);
(gdb) x/16xb bad_string
0x7fff5fbff9cd:	0x63	0x61	0x74	0xe0	0xf9	0xbf	0x5f	0xff
0x7fff5fbff9d5:	0x7f	0x00	0x00	0xfd	0xb5	0x23	0x89	0xff
```

Here, by using the `x` command with parameters `16xb`, we can see that starting at memory address `0x7fff5fbff9c` (value of `bad_string`), printf would actually see the following sequence of bytes as a string because we provided a malformed string without a null terminator.

```0x43 0x61 0x74 0xe0 0xf9 0xbf 0x5f 0xff 0x7f 0x00```