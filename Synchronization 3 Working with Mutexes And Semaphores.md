## What is an atomic operation?
To paraphrase Wikipedia, "An operation (or set of operations) is atomic or uninterruptible if it appears to the rest of the system to occur instantaneously."
Without locks, only simple CPU instructions ("read this byte from memory") are atomic (indivisible). 

Incrementing a variable (`i++`) is not atomic because it requires three distinct steps: Copying the bit pattern from memory into the CPU; performing a calculation using the CPU's registers; copying the bit pattern back to memory. During this increment sequence, another thread or process can still read the old value and other writes to the same memory would also be over-written when the increment sequence completes.


## How do I use mutex lock to make my data-structure thread-safe?
Note, this is just an introduction - writing high-performance thread-safe data-structures requires it's own book! Here's a simple data structure (a stack) that is not thread safe:
```C
// A simple fixed-sized stack (version 1)
int count;
double values[count];

void push(double v) { values[count++] = v; }
double pop() { return values[--count]; }
int is_empty() { return count == 0; }
```
Version 1 of the stack is not thread-safe because if two threads call push or pop at the same time then the results or the stack can be inconsistent. For example, imagine if two threads call pop at the same time then both threads may read the same value, both may read the original count value.

To turn this into a thread-safe data structure we need to identify the _critical sections_ of our code  i.e. which section(s) of the code must only have one thread at a time. In the above example the `push`,`pop` and `is_empty` functions access the same variables (i.e. memory) and all critical sections for the stack. 

While `push` (and `pop`) is executing, the datastructure is an inconsistent state (for example the count may not have been written to, so may still contain the original value). By wrapping these methods with a mutex we can ensure that only one thread at a time can update (or read) the stack.

A candidate 'solution' is shown below. Is it correct? If not, how will it fail?
```
// An attempt at a thread-safe stack (version 2)
int count;
double values[count];
pthread_mutex_t m1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t m2 = PTHREAD_MUTEX_INITIALIZER;

void push() { pthread_mutex_lock(&m); values[count++] = values; pthread_mutex_unlock(&m); }
double pop() { pthread_mutex_lock(&m2); double v=values[--count]; pthread_mutex_unlock(&m2); return v;}
int is_empty() { pthread_mutex_lock(&m); return count == 0; pthread_mutex_unlock(&m); }
```
The above code ('version 2') contains at least one error. Take a moment to see if you can the error(s) and work out the consequence(s).

If three called `push()` at the same time the lock `m` ensures that only one thread at time manipulates the stack (two threads will need to wait until the first thread completes (calls unlock), then a second thread will be allowed to continue into the critical section and finally the third thread will be allowed to continue once the second thread has finished).

A similar argument applies to concurrent calls (calls at the same time) to `pop`. However version 2 does not prevent push and pop from running at the same time because `push` and `pop` use two different mutex locks.

The fix is simple in this case - use the same mutex lock for both the push and pop functions.

The code has a second error; `is_empty` returns after the comparison and will not unlock the mutex. However the error would not be spotted immediately. For example, suppose one thread calls `is_empty` and a second thread later calls `push`. This thread would mysteriously stop. Using debugger you can discover that the thread is stuck at the lock() method inside the `push` method because the lock was never unlocked by the earlier `is_empty` call. Thus an oversight in one thread led to problems much later in time in an arbitrary other thread.

A better version is shown below - 
```
// An attempt at a thread-safe stack (version 3)
int count;
double values[count];
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void push(double v) { pthread_mutex_lock(&m); values[count++] = v; pthread_mutex_unlock(&m); }
double pop() { pthread_mutex_lock(&m); double v= values[--count]; pthread_mutex_unlock(&m); return v;}
int is_empty() { pthread_mutex_lock(&m); int result= count == 0; pthread_mutex_unlock(&m);return result; }
```
Version 3 is thread-safe (we have ensured mutual exclusion for all of the critical sections) however there are two points of note:
* `is_empty` is thread-safe but its result may already be out-of date i.e. the stack may no longer be empty by the time the thread gets the result!
* There is no protection against underflow (popping on an empty stack) or overflow (pushing onto a an already-full stack)

The latter point can be fixed using counting semaphores.

The implementation assumes a single stack.  A more general purpose version might include the mutex as part of the memory struct and use pthread_mutex_init to initialize the mutex. For example,

```
struct stack {
  int count;
  pthread_mutex_t m; 
  double* values;
};
typedef struct stack stack_t

stack_t* stack_create(int capacity) {
  stack_t* result = malloc( sizeof(stack_t) );
  result -> values = malloc( sizeof(double) * capacity );
  pthread_mutex_init(& result->m, NULL);
  return result;
}
void stack_destroy(stack_t*s) {
  free(s->values);
  pthread_mutex_destroy(& s->m);
  free(s);
}
void push(stack_t*s, double v) { pthread_mutex_lock(&s->m); s->values[(s->count)++] = v; pthread_mutex_unlock(&s->m); }
double pop(stack_t*s) { pthread_mutex_lock(&s->m); double v= s->values[-- (s->count)]; pthread_mutex_unlock(&s->m); return v;}
int is_empty(stack_t*s) { pthread_mutex_lock(&s->m); int result= s->count == 0; pthread_mutex_unlock(&s->m);return result; }

```
## When can I destroy the mutex?
You can only destroy an unlocked mutex

## Can I copy a pthread_mutex_t to a new memory locaton?
No, copying the bytes of the mutex to a new memory location and then using the copy is _not_ supported.

## What would a simple implementation of a mutex look like?

A simple (but incorrect!) suggestion is shown below. The `unlock` function simply unlocks the mutex and returns. The lock function first checks to see if the lock is already locked. If it is currently locked, it will keep checking again until another thread has unlocked the mutex.
```
// Version 1 (Incorrect!)

void lock(mutex_t* m) {
  while(m->locked) { /*Locked? Nevermind - just loop and check again!*/ }

  m->locked = 1;
}
void unlock(mutex_t*m) {
  m->locked = 0;
}
```
Version 1 uses 'busy-waiting' (unnecessarily wasting CPU resources) however this is a more serious problem: We have a race-condition! If two threads both called `lock` concurrently it is possible that both threads would read 'm_locked' as zero. Thus both threads would believe they have exclusive access to the lock and both threads will continue. Ooops!

We could reduce the CPU overhead a little by calling `pthread_yield` inside the loop (this suggests to the operating system that the thread does not the CPU for a while, so the CPU may be assigned to threads that are waiting to run) but does not fix the race-condition. We need a better implementation - can you work out one?


## Mutex Gotcha
* Not unlocking a mutex (due to say an early return during an error condition)
* Resource leak (not calling mutex_destroy)
* Using an unitialized mutex (or using a mutex that has already been destroyed)
* Locking a mutex twice on a thread (without unlocking first)
