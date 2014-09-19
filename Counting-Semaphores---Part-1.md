## What is a counting semaphore?
A counting semaphore contains a value and supports two operations "wait" and "post". Post increments the semaphore and immediately returns. "wait" will wait if the count is zero. If the count is non-zero the semaphore decrements the count and immediately returns.

An analogy is a count of the cookies in a cookie jar (or gold coins in the treasure chest). Before taking a cookie, call 'wait'. If there are no cookies left then `wait` will not return: It will `wait` until another thread increments the semaphore by calling post.

In short, `post` increments and immediately returns whereas `wait` will wait if the count is zero. Before returning it will decrement count.

## Can I call wait and post from different threads?
Yes! Unlike a mutex, the increment and decrement can be from different threads.

## Can I use a semaphore instead of a mutex?
Yes - though the overhead of a semaphore is greater. To use a semaphore:
* Initialize the semaphore with a count of one.
* Replace `...lock` with `sem_wait`
* Replace `...unlock` with `sem_post`

## Can I use sem_post inside a signal handler?
Yes! `sem_post` is one a handful of functions that can be correctly used inside a signal handler.

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
    sem_init(&s, 0, 0 /* Initial value of zero*/); 
    signal(SIGINT, handler);

    pthread_t tid;
    pthread_create(&tid, NULL, singsong, NULL);
    pthread_exit(NULL); /* Process will exit when there are no more threads */
}
```


