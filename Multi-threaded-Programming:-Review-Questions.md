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
Complete the following code. Add condition variable calls to `func` so that the waiting thread does not need to continually check the `turn` variable.
````C
pthread_cond_t cv = PTHREAD_COND_INITIALIZER;
pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;

void* turn;

void* func(void*mesg) {
  while(1) {
    while(turn == mesg) { /* poll again ... Fix me - Wastes lots of CPU time!*/ }
    puts( (char*) mesg);
    turn = mesg;
  }
  return 0;
}

int main(int argc, char**argv){
  pthread_t tid1;
  pthread_create(&tid1,NULL, func, "A");
  func("B");
  return 0;
}
````