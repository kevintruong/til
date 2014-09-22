## What is an atomic operation?
Todo

## How do I use mutex lock to make my data-structure thread-safe?
Here's a simple data structure (a stack) that is not thread safe:
```C
// A stack (version 1)
int count;
double values[count];

void push() { values[count++] = values; }
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

void push() { pthread_mutex_lock(&m); values[count++] = values; pthread_mutex_unlock(&m); }
double pop() { pthread_mutex_lock(&m); double v= values[--count]; pthread_mutex_unlock(&m); return v;}
int is_empty() { pthread_mutex_lock(&m); int result= count == 0; pthread_mutex_unlock(&m);return result; }
```
Version 3 is thread-safe (we have ensured mutual exclusion for all of the critical sections) however there are two points of note:
* `is_empty` is thread-safe but its result may already be out-of date i.e. the stack may no longer be empty by the time the thread gets the result!
* There is no protection against underflow (popping on an empty stack) or overflow (pushing onto a an already-full stack)

The latter point can be fixed using counting semaphores.

## A simple but incorrect mutex lock implementation

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









## Gotcha
Early returns. F