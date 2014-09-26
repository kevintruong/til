## Warm up

* "Only one process(/thread) can be in the CS at a time"
* "If waiting, then another process can only enter the CS a finite number of times" 
* "If no other process is in the CS then the process can immediately enter the CS"
See [[Synchronization-4-The-Critical-Section-Problem]] for answers.

## What is the 'exchange instruction' ?
The exchange instruction ('XCHG') is an atomic CPU instruction that exchanges the contents of a register with a memory location. This can be used as a basis to implement a simple mutex lock.
```C
// Psuedo-C-code for a simple busy-waiting mutex 
// that uses an atomic exchange function
int lock = 0; // initialization

// To enter the critical section you need to read a lock value of zero. 
while( xchg( 1, &lock) ) {/*spin spin spin*/}
/* Do Critical Section stuff*/
lock = 0;
```

What are condition variables? How do you use them? What is Spurious Wakeup?

* Condition variables allow a set of threads to sleep until tickled! You can tickle one thread or all threads that are sleeping. If you only wake one thread then the operating system will decide which thread to wake up. You don't wake threads directly instead you 'signal' the condition variable, which then will wake up one (or all) threads that are sleeping inside the condition variable.

* Condition variables are used with a mutex and with a loop (to check a condition).

* Occasionally a waiting thread may appear to wake up for no reason (this is called a _spurious wake_)! This is not an issue because you always use `wait` inside a loop that tests a condition that must be true to continue.

## What does pthread_cond_wait do?
Wait performs three actions:
* unlock the mutex
* waits (sleeps until pthread_cond_signal is called on the same condition variable)
* locks the mutex

## Why do spurious wakes exist?
For performance. On multi-CPU systems it is possible that a race-condition could cause a wake-up (signal) request to be unnoticed. The kernel may not detect this lost wake-up call but can detect when it might occur. To avoid the potential lost signal the thread is woken up so that the program code can test the condition again.

## Example
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
   pthread_cond_wait(&cv,&m); /*unlock m,wait, lock m*/
}
pthread_mutex_unlock(&m);

//later 
pthread_cond_destroy(&cv,); 
```

An another thread can increment count:
while(1) {
  pthread_mutex_lock(&m);
  count ++;
  pthread_cond_signal(&cv);
  /* Even though the other thread is woken up it will not return
  we unlock the mutex */
  pthread_mutex_unlock(&m);
}
```
U



