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
In the above example, we allocate enough bytes to hold a pointer to a user struct.

## Using uninitialized variables
```C
int result;
result += 10;
```
## Assuming Uninitialized memory will be zeroed
```C
void myfunct() {
   char array[10];
   char *p = malloc(10);
```
## Double-free
## Dangling pointers

# Logic and Program flow mistakes

## Forgetting break
## Equal vs equality
## Undeclared Functions
## Extra Semicolons

# Other mistakes
## C Preprocessor 
