## The introduction

The reader-writer problem appears in many different contexts. For example a web server cache needs to quickly serve the same static page for many thousands of requests. Occasionally however an author may decide to update the page.

We will examine one specific form of the reader-writer problem where there are many readers and some occasional writers and we need to ensure that a writer gets exclusive access. For performance however readers should be able to perform the read without waiting for another reader. 

See [[Synchronization,-Part-7:-The-Reader-Writer-Problem]] for part 1

## Candidate solution #3
Candidate solutions 1 and 2 are discussed in [[part 1|Synchronization,-Part-7:-The-Reader-Writer-Problem]].


In the code below for clarity `lock` and `cond_wait` are shortened to `pthread_mutex_lock` and `pthread_cond_wait` etc.

Also remember that `pthread_cond_wait` performs *Three* actions. Firstly it atomically unlocks the mutex and then sleeps (until it is woken by `pthread_cond_signal` or `pthread_cond_broadcast`). Thirdly the awoken thread must re-acquire the mutex lock before returning. Thus only one thread can actually be running inside the critical section defined by the lock and unlock() methods.

Implementation #3 below ensures that a reader will enter the cond_wait if there are any writers writing.
```C
read(){
 lock(&m)
 while (writing)
    cond_wait(&turn, &m)
 reading++;

/* Read here! */

 reading--
 cond_signal(&turn)
 unlock(&m)
```
However only one reader a time can read because candidate #3 did not unlock the mutex. A better version unlocks before reading :
```C
read(){
 lock(&m);
 while (writing)
    cond_wait(&turn, &m)
 reading++;
 unlock(&m)
/* Read here! */
 lock(&m)
 reading--
 cond_signal(&turn)
 unlock(&m)
```
Does this mean that a writer and read could read and write at the same time? No! First of all, remember cond_wait requires the thread re-acquire the  mutex lock before returning. Thus only one thread can be executing code inside the critical section (marked with **) at a time!

read(){
 lock(&m);
**  while (writing)
**    cond_wait(&turn, &m)
**  reading++;
 unlock(&m)
/* Read here! */
 lock(&m)
**  reading--
**  cond_signal(&turn)
  unlock(&m)
</pre>


Writers must wait for everyone. Mutual exclusion is assured by the lock. 
```C
write(){
 lock(&m);
**   while (reading || writing)
**     cond_wait(&turn, &m);
**   writing++;
**
** /* Write here! */
**  writing--;
**  cond_signal(&turn);
  unlock(&m);
```

Todo : Discuss this candidate