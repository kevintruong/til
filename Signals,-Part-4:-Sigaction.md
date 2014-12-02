## How and why do I use `sigaction` ?

To change the "signal disposition" of a process - i.e. what happens when a signal is delivered to your process - use `sigaction`

```C
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);

struct sigaction {
               void     (*sa_handler)(int);
               void     (*sa_sigaction)(int, siginfo_t *, void *);
               sigset_t   sa_mask;
               int        sa_flags;
}; 
```

struct sigaction sa; 
sa.sa_handler = handler;
sigemptyset(&sa.sa_mask);   //Also  sigfillset
sa.sa_flags = SA_RESTART; /* Restart functions if  interrupted by handler */     
sigaction(SIGINT, &sa, NULL)


(Under construction)