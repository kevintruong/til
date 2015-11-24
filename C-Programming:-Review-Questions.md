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

typedef ______________________;


In addition to the function arguments what else is stored on a thread's stack?
