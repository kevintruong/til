## How do print something out in C to the console? 
Use `printf`. The first parameter is a format string that includes placeholders for the data to be printed. Common format specifiers are `%s` treat the argument as a c string pointer, keep printing all characters until the NULL-character is reached; `%d` print the argument as an integer; `%p` print the argument as a memory address.
A simple example is shown below:
```C
char *name = ... ; int score = ...;
printf("Hello %s, your result is %d\n", name, score);
printf("Debug: The string and int are stored at: %p and %p\n", name, &score );
// name already is a char pointers. We use "&" to get the address of the int variable
```

By default, for performance, `printf` does not actually write anything out (by calling write) until its buffer is full or a newline is printed. 

## How else can I print strings and single characters?
Use `puts( name );` and `putchar( c )`  wher name is a pointer to a C string and c is just a `char`

## How do I print to other file streams?
Use `fprintf( _file_ , "Hello %s, score: %d", name, score);`
Where \_file\_ is either predefined 'stdout' 'stderr' or a FILE pointer that was returned by `fopen` or `fdopen`

## How do I print data into a C string?
Use `sprintf` or better `snprintf`.
```C
char result[200];
int len = snprintf(result, sizeof(result) , "%s:%d",name,score);
```
snprintf returns the number of characters written excluding the terminating byte. In the above example this would be a maximum of 199.

## How do I parse input using scanf into parameters?
Use `scanf` (or fscanf or sscanf) to get input from the default input stream, an arbitrary file stream or a C string respectively.
It's a good idea to check the return value to see how many items were parsed.
scanf requires valid pointers. It's a common source of error to pass in an incorrect pointer value. For 
example,
```
int* data = (int*) malloc(sizeof(int));
char* line = "v 10";
char type;
sscanf(line,"%c %d", &type, &data); // oops!
```
We wanted to write the character value into c and the integer value into the malloc'd memory.
However we passed the address of the pointer, not what the pointer is pointing to. So scanf will change the pointer. i.e. the pointer will now point to address 10 so this code will later fail e.g. when free(data) is called.
 
## Why is gets dangerous? What should I use intead.
The following code is vulnerable to buffer overflow. It assumes or trusts that the input line will be no more than 10 characters, including the terminating byte.
```
char buf[10];
gets(buf); // Remember the array name means the first byte of the array
``` 
gets is now deprecated and will be removed in future versions of the C standard. Programs should use fgets or getline instead. 

`char * fgets ( char * str, int num, FILE * stream); `
`ssize_t getline(char **lineptr, size_t *n, FILE *stream);`

Here's a simple, safe way to read a single line. Lines longer than 9 characters will be truncated:
```C
char buffer[10];
char* result =  fgets(result, sizeof(buffer), stdin);
```
The result is NULL if there was an error or the end of the file is reached.
Note fgets copies the newline into the buffer, which you may want to discard-
```
int len= strlen(buffer);
for(; len>=0 ; len--) {
  if (buffer[len] == '\n')  {
     buffer[len] = '\0';  /* Truncate */ 
     break;
  }
}
```
One of the advantages of getline is that will automatically (re-) allocate a buffer of sufficient size.
