Todo: Move this into Sync 4:
## Name these concepts:
* "Only one process(/thread) can be in the CS at a time"
* "If waiting, then another process can only enter the CS a finite number of times" 
* "If no other process is in the CS then the process can immediately enter the CS"

## Todo : Explain why implementing a correct solution might fail.
## Todo : Peterson, Dekker solutions

## What is the 'exchange instruction' ? Why is it useful


What are condition variables? How do you use them? What is Spurious Wakeup?

* Condition variables allow a set of threads to sleep until tickled! You can tickle one thread or all threads. If you only wake one thread then the operating system will decide which thread to wake up.

* Condition variables are used with a mutex and with a loop (to check some condition).

* Occasionally a waiting thread may appear to wake up for no reason (this is called a spurious wake)! This is not a problem because you always use `wait` inside a loop that tests a condition that must be true to continue.

## Why do spurious wakes exist?
For performance. On multi-CPU systems it is possible that a race-condition could cause a wake-up request to be unnoticed. The kernel may not detect this lost wake-up call but can detect when it might occur. If so, the thread is woken up so that the program code can test the condition again.

## How do I use POSIX condition variables
Condition variables are _always_ used with a mutex lock.
Before calling wait, the mutex lock must be locked.
```C
pthread_cond_t cv;
pthread_mutex_t m;
int count;

// Initialize
pthread_cond_init(&cv,NULL);
pthread_mutex_init(&m,NULL);
count = 0;

pthread_mutex_lock(&m);
while( count < 10 ){
   pthread_cond_wait(&cv,&m);
}
pthread_mutex_unlock(&m);

//later 
pthread_cond_destroy(&cv,); 
```

Another thread can increment count:
while(1) {
  pthread_mutex_lock(&m);
  count ++;
  pthread_cond_signal(&cv);
  pthread_mutex_unlock(&m);
}


```
Under construction!




