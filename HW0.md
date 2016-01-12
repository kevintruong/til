## Answer These Questions As You Watch These Videos: http://cs-education.github.io/sys/

### Chapter 1
- Hello World (System call style)
  - Write me a program that uses write() to print out "Hi! My name is <Your Name>".
- Hello Standard Error Stream
  - Write me a program that uses write() to prints out a triangle of height n to Standard Error
    - n should be a variable and the triangle should look like this for n = 3
    ```
    *
    **
    ***
    ```
- Writing to files
  - Take your program from "Hello World" and have it write to a file
    - Make sure to to use some interesting flags and mode for open()
    - ```man 2 open``` is your friend
- Not everything is a system call
  - Take your program from "Writing to files" and replace it with printf()
  - Name some differences from write() and printf()

### Chapter 2
- Not all bytes are 8 bits?
  - How many bits are there in a byte?
  - How many bytes is a char?
  - Tell me how many bytes the following are on your machine: int, double, float, long, long long
- Follow the int pointer
  - On a machine with 8 byte integers:
  ```C
  int main(){
      int data[8];
  } 
  ```
  If the address of data is 0x7fbd9d40, then what is the address of data+2?
  - What is data[3] equivalent to in C?
- sizeof character arrays, incrementing pointers
  - Why does this segfault?
  ```C
  char *ptr = "hello";
  *ptr = 'J';
  ```
  - What does sizeof("Hello\0World") return?
  - What does strlen("Hello\0World") return?

### Chapter 3
- Program arguments argc argv
  - Name me two ways to find the length of argv
  - What is argv[0]
- Environment Variables
  - Where are the pointers to environment variables stored?
- String searching (Strings are just char arrays)
  - On a machine where pointers are 8 bytes and with the following code:
  ```C
  char *ptr = "Hello";
  char array[] = "Hello";
  ```
  What is the results of sizeof(ptr) and sizeof(array);
- Lifetime of automatic variables
  - What datastucture is managing the lifetime of automatic variables?

### Chapter 4
- Malloc, heap and time
  - If I want to use data after the lifetime of the function it was created in, then where should I put it and how do I put it there?
  - Fill in the blank. In a good C program: "For every malloc there is a ___".
- Heap allocation Gotchas
  - Name one reason malloc can fail.
  - Name some differences between time() and ctime()
  - What is wrong with this code snippet?
  ```C
  free(ptr);
  free(ptr);
  ```
  - What is wrong with this code snippet?
  ```C
  free(ptr);
  printf("%s\n", ptr);
  ```
  - How can one avoid the previous 2 mistakes? 
- struct, typedefs and a linked list
  - Write me a struct that represents a Person and typedef, so I do not have to write "struct Person" all over the place
    - A person should contain the following information: Name, Age, Friends (array of people).
  - Now make a person "Bob" on the heap who is 40 years old and his only friend is himself :(
- Duplicating strings, memory allocation and deallocation of structures
  - Write me functions to create and destroy a Person.
    - create() should take a name and make a copy of it and also an age
    - destroy() should free up not only the memory to bookkeep the person but also all its attributes.

### Chapter 5 
- Reading characters, Trouble with gets
  - What functions can be used for getting characters for stdin and writing them to stdout?
  - Name one issue with gets()
- Introducing sscanf and friends
  - Write me code that takes the string "Hello 5 World" and break it up into 3 parts ("Hello", 5, "World")
- getline is useful
  - What does one need to define before using getline()?
  - Write me a C program to print out the content of a file line by line using getline()