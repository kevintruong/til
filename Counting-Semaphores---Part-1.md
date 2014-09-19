## What is a counting semaphore?
A counting semaphore contains a value and supports two operations "wait" and "post". Post increments the semaphore and immediately returns. "wait" will wait if the count is zero. If the count is non-zero the semaphore decrements the count and immediately returns.

An analogy is a count of the cookies in a cookie jar (or gold coins in the treasure chest). Before taking a cookie, call 'wait'. If there are no cookies left then `wait` will not return: It will `wait` until another thread increments the semaphore by calling post.

In short, `post` increments and immediately returns whereas `wait` will wait if the count is zero. Before returning it will decrement count.

## How do I create a semaphore?
This page introduces unnamed semaphores. Unfortunately Mac OSX does not support these yet.

First decide if the initial value should be zero or some other value (e.g. the number of remaining spaces in an array).
Unlike pthread mutex there are not shortcuts to creating a semaphore - use 'sem_init'
```C
sem_t s;
int main() {
  sem_init( &s , 0, 10); // returns -1 (=FAILED) on OSX
  sem_wait(&s); // Could do this 10 times without blocking
  sem_post(&s); // Announce that we've finished (and one more resource item is available; increment count)
  sem_destroy(&s); // release resources of the semaphore
```

## Can I call wait and post from different threads?
Yes! Unlike a mutex, the increment and decrement can be from different threads.

## Can I use a semaphore instead of a mutex?
Yes - though the overhead of a semaphore is greater. To use a semaphore:
* Initialize the semaphore with a count of one.
* Replace `...lock` with `sem_wait`
* Replace `...unlock` with `sem_post`

## Can I use sem_post inside a signal handler?
Yes! `sem_post` is one a handful of functions that can be correctly used inside a signal handler.
This means we can release a waiting thread which can now make all of the calls that we were not
allowed to call inside the signal handler itself (e.g. printf).

```C
#include <stdio.h>
#include <pthread.h>
#include <signal.h>
#include <semaphore.h>
#include <unistd.h>

sem_t s;

void handler(int signal)
{
    sem_post(&s);
}

void *singsong(void *param)
{
    sem_wait(&s);
    printf("I had to wait until your signal released me!\n");
}

int main()
{
    int ok = sem_init(&s, 0, 0 /* Initial value of zero*/); 
    if(ok == -1) {
       perror("Could not create unnamed semaphore");
       return 1;
    }
    signal(SIGINT, handler);

    pthread_t tid;
    pthread_create(&tid, NULL, singsong, NULL);
    pthread_exit(NULL); /* Process will exit when there are no more threads */
}
```


