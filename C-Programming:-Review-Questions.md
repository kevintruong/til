## Memory and Strings
In the example below, which variables are guaranteed to print the value of zero?
````C
int a;
static int b;

void func() {
   static int c;
   int d;
   printf("%d %d %d %d\n",a,b,c,d);
}
````

In the example below, which variables are guaranteed to print the value of zero?
````C
void func() {
   int* ptr1 = malloc( sizeof(int) );
   int* ptr2 = realloc(NULL, sizeof(int) );
   int* ptr3 = calloc( 1, sizeof(int) );
   int* ptr4 = calloc( sizeof(int) , 1);
   
   printf("%d %d %d %d\n",*ptr1,*ptr2,*ptr3,*ptr);
}
````

Explain the error in the following attempt to copy a string.
````C
char* copy(char*src) {
 char*result = malloc( strlen(src) ); 
 strcpy(result, src); 
 return result;
}
````

Why does the following attempt to copy a string sometimes work and sometimes fail?

````C
char* copy(char*src) {
 char*result = malloc( strlen(src) +1 ); 
 strcat(result, src); 
 return result;
}
````

Explain the two errors in the following code that attempts to copy a string.
````C
char* copy(char*src) {
 char result[sizeof(src)]; 
 strcpy(result, src); 
 return result;
}
````

Which of the following is legal?
````C
char a[] = "Hello"; strcpy("World", a);
char b[] = "Hello"; strcpy("World12345", b);
char* c = "Hello"; strcpy("World", c);
````

Complete the function pointer typedef to declare a pointer to a function that takes a void* argument and returns a void*. Name your type 'pthread_callback'
````C
typedef ______________________;
````

In addition to the function arguments what else is stored on a thread's stack?


## Printing

Spot the error!
````C
fprintf(stdout, "You scored 100%");
````
## Formatting - printing to a file

Complete the following code to print to a file. Print the name, a comma and the score to the file 'result.txt'
````C
char* name = .....;
int score = ......
FILE *f = fopen("result.txt",_____);
if(f) {
    _____
}
fclose(f);
````
## Formatting - printing to a string

How would you print the values of variables a,mesg,val and ptr to a string? Print a as an integer, mesg as C string, val as a double val and ptr as a hexadecimal pointer. You may assume the mesg points to a short C string(<50 characters).
Bonus: How would you make this code more robust or able to cope with?
```C
char* toString(int a, char*mesg, double val, void* ptr) {
   char* result = malloc( strlen(mesg) + 50);
    _____
   return result;
}

## Input. Parsing
Why should you check the return value of sscanf and scanf?

Why is 'gets' dangerous?

Write a complete program that uses `getline`

## Heap memory
When would you use calloc not malloc? 
When would realloc be useful?
