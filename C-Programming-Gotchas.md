#What common mistakes do C programmers make?
For each of these entries we need some code, an explanation, and suggestions on how to avoid the mistake.

# Memory mistakes
## Array boundaries
```C
#define N (10)
int i=N, array[N];
for( ; i>=0; i--) array[i]=i;
```
C does not check that pointers are valid. The above example writes into `array[10]` which is outside the array bounds. This can cause memory corruption because that memory location is probably being used for something else.

## Insufficient memory allocation 
```C
struct User {
   char name[100];
};
typedef struct User user_t;

user_t* user = (user_t*) malloc(sizeof(user));
```
In the above example, we needed to allocate enough bytes for the struct. Instead we allocated enough bytes to hold a pointer. Once we start using the user pointer we will corrupt memory.

## Using uninitialized variables
```C
int myfunction() {
  int x;
  int y = x + 2;
...
```
Automatic variables hold garbage (whatever bit pattern happened to be in memory). It is an error to assume that it will always be initialized to zero.

## Assuming Uninitialized memory will be zeroed
```C
void myfunct() {
   char array[10];
   char *p = malloc(10);
```
## Double-free
```C
  char *p = malloc(10);
  free(p);
//  .. later ...
  free(p); 
```
It is an error to free the same block of memory twice.
## Dangling pointers
```C
  char *p = malloc(10);
  strcpy(p,"Hello");
  free(p);
//  .. later ...
  strcpy(p,"World"); 
```
Pointers to freed memory should not be used. A defensive programming practice is to set pointers to null as soon as the memory is freed.

# Logic and Program flow mistakes
## Forgetting break
```C
int flag = 1; // Will print all three lines.
switch(flag) {
  case 1: printf("I'm printed\n");
  case 2: printf("Me too\n");
  case 3: printf("Me three\n");
}
```
Case statements without a break will just continue onto the code of the next case statement.

## Equal vs equality
```C
int answer = 3; // Will print out the answer.
if(answer=42) { printf("I've solved the answer! It's %d", answer);}
```

## Undeclared Functions
## Extra Semicolons
```C
for(int i = 0; i < 5; i++) ; printf("I'm printed once");
while(x < 10 ) ; x++ ; // X is never incremented
```

# Other Gotchas
## C Preprocessor macros and side-effects
```C
#define min(a,b) ((a)<(b) ? (a) : (b))
int x = 4;
if(min(x++, 100)) printf("%d is six", x);
```
Macros are simple text substitution so the above example expands to `x++ < 100 ? x++ : 100` (parenthesis omitted for clarity)

## C Preprocessor macros and precedence
```C
#define min(a,b) a<b ? a : b
int x = 99;
int r = 10 + min(99,100); // r is 100!
```
Macros are simple text substitution so the above example expands to `10 + 99 < 100 ? 99 : 100`