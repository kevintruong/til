## What is the critical section problem?

As already discussed [here] (todo link), there are critical parts of our code that can only be executed by one thread at a time. We describe this requirement as 'mutual exclusion'; only one thread (or process) may have access to the shared resource.

In multi-threaded programs we can wrap a critical section with mutex lock and unlock calls:

```C
pthread_mutex_lock() - one thread allowed at a time! (others will have to wait here)
... Do Critical Section stuff here!
pthread_mutex_unlock() - let other waiting threads continue
```
How would we implement these lock and unlock calls? Can we create algorithm that assures mutual exclusion. An incorrect implementation is shown below, 
```C
pthread_mutex_lock(p_mutex_t* m)     { while(m->lock) {}; m->lock = 1;}
pthread_mutex_unlock(p_mutex_t* m)   { m->lock = 0; }
```


At first glance, the code appears to work; if one thread attempts to locks the mutex, a later thread must wait until the lock is cleared. However this implementation _does not satisfy Mutual Exclusion_. Let's take a close look at this 'implementation' from the point of view of two threads running around the same time. In the table below times runs from top to bottom-

Time | Thread 1 | Thread 2
-----|----------|---------
1 | `while(lock) {}`
2 | | `while(lock) {} ` | 
3 | lock = 1 | lock = 1 |
Ooops! There is a race condition. In the unfortunate case both threads checked the lock and read a false value and so were able to continue. 

## Candidate solutions to the critical section problem.
To simplify the discussion we imagine only two threads. Note the arguments work for both threads and processes and the classic CS literature discusses these problem in terms of two processes that need exclusive access (i.e. mutual exclusion) to access a shared resource.

Remember that the psuedo-code outlined below is part of a larger program; the thread or process will typically need to enter the critical section many times during the lifetime of the process. So imagine each example as wrapped inside a loop where for a random amount of time the thread or process is working on something else.

Is there anything wrong with candidate solution described below?
```
// Candidate #1
wait until your flag is lowered
raise my flag
// Do Critical Section stuff
lower my flag 
```

Answer: Candidate solution #1 also suffers a race condition i.e. it does not satisfy Mutual Exclusion because both threads/processes could read each other's flag value (=lowered) and continue. 

This suggests we should raise the flag _before_ checking the other thread's flag - which is candidate solution #2 below.

```
// Candidate #2
raise my flag
wait until your flag is lowered
// Do Critical Section stuff
lower my flag 
```

Candidate #2 satisfies mutual exclusion - it is impossible for two threads to be inside the critical section at the same time. However this code suffers from deadlock! Suppose two threads wish to enter the critical section at the same time:

Time | Thread #1 | Thread #2
-|-|-
1 | raise flag |
2 | | raise flag |
3 | wait ... | wait ...
Ooops both threads / processes are now waiting for the other one to lower their flags. Neither one will enter the critical section as both are now stuck forever!

This suggests we should use a turn-based variable to try to resolve who should proceed. 

## Turn-based solutions
The following candidate solution #3 uses a turn-based variable to politely allow one thread and then the other to continue

```
// Candidate #3
wait until my turn is myid
// Do Critical Section stuff
turn = yourid
```
Candidate #3 satisfies mutual exclusion (each thread or process gets exclusive access to the Critical Section), however both threads/processes must take a strict turn-based approach to using the critical section. For example, if thread 1 wishes to read a hashtable every millisecond but another thread writes to a hashtable every second, then the reading thread would have to wait another 999ms before being able to read from the hashtable again. This 'solution' is not effective, because our threads should be able to make progress if no other thread is currently in the critical section.

## Desired properties for solutions to the Critical Section Problem?
There are three main desirable properties that we desire in a solution the critical section problem
* Mutual Exclusion - the thread/process gets exclusive access; others must wait until it exits the critical section.
* Bounded Wait - if the thread/process has to wait, then it should only have to wait for a finite, bounded amount of time (infinite waiting times are not allowed!)
* Progress - if no thread/process is inside the critical section, then the thread/process should be able to proceed (make progress) without having to wait.


With these ideas in mind let's examine another solution that uses a turn-based flag only if two threads both required access at the same time.

## Turn and Flag solutions



