## Warning - question numbers subject to change
## Q1
Is the following code thread-safe? Redesign the following code to be thread-safe. Hint: A mutex is unnecessary if the message memory is unique to each call.

````C
static char message[20];
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZAER;

void format(int v) {
  pthread_mutex_lock(&mutex);
  sprintf(message,":%d:",v);
  pthread_mutex_unlock(&mutex);
  return message;
}
````
## Q2
Which one of the following does not cause a process to exit?
* Returning from the pthread's starting function in the last running thread.
* The original thread returning from main.
* Any thread causing a segmentation fault.
* Any thread calling `exit`.
* Calling `pthread_exit` in the main thread with other threads still running.


## Q3
Write a mathematical expression for the number of "W" characters that will be printed by the following program. Assume a,b,c,d are small positive integers. Your answer may use a 'min' function that returns its lowest valued argument.

````C
unsigned int a=...,b=...,c=...,d=...;

void* func(void* ptr) {
  char m = * (char*)ptr;
  if(m == 'P') sem_post(s);
  if(m == 'W') sem_wait(s);
  putchar(m);
  return NULL;
}

int main(int argv, char**argc) {
  sem_init(s,0, a);
  while(b--) pthread_create(&tid,NULL,func,"W"); 
  while(c--) pthread_create(&tid,NULL,func,"P"); 
  while(d--) pthread_create(&tid,NULL,func,"W"); 
  pthread_exit(NULL); 
  /*Process will finish when all threads have exited */
}
````

## Q4
Complete the following code. The following code is supposed to print alternating `A` and `B`. It represents two threads that take turns to execute.  Add condition variable calls to `func` so that the waiting thread does not need to continually check the `turn` variable. Hint: Why is cond_broadcast necessary?
````C
pthread_cond_t cv = PTHREAD_COND_INITIALIZER;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void* turn;

void* func(void*mesg) {
  while(1) {
// Add mutex lock and condition variable calls ...

    while(turn == mesg) { /* poll again ... Change me - This busy loop burns CPU time!*/ }

    /* Do stuff on this thread*/
    puts( (char*) mesg);
    turn = mesg;
    
  }
  return 0;
}

int main(int argc, char**argv){
  pthread_t tid1;
  pthread_create(&tid1,NULL, func, "A");
  func("B"); // no need to create another thread - just use the main thread
  return 0;
}
````

## Q5
Identify the critical sections in the given code. Add mutex locking to make the code thread safe. Add condition variable calls so that `total` never becomes negative or above 1000. Instead the call should block until it is safe to proceed. Explain why `pthread_cond_broadcast` is necessary.
````C
int total;
void add(int value) {
 if(value) <1) return;
 total += value;
}
void sub(int value) {
 if(value) <1) return;
 total -= value;
}
````
## Q5
Identify the critical sections in the given code. Add mutex locking to make the code thread safe. Add condition variable calls so that `total` never becomes negative or above 1000. Instead the call should block until it is safe to proceed. 
````C
int total;
void add(int value) {
 if(value) <1) return;
 total += value;
}
void sub(int value) {
 if(value) <1) return;
 total -= value;
}
````

## Q6
A non-threadsafe datastructure has `size()` `enq` and `deq` methods. Use condition variable and mutex lock to complete the thread-safe, blocking versions.
````C
void enqueue(void* data) {
// should block if the size() > 256
enq(data);
}
void* dequeue() {
// should block if size() is 0
return deq();
}