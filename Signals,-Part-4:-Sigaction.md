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

```C
struct sigaction sa; 
sa.sa_handler = handler;
sigemptyset(&sa.sa_mask);   //Also  sigfillset
sa.sa_flags = SA_RESTART; /* Restart functions if  interrupted by handler */     
sigaction(SIGINT, &sa, NULL)
```

## Full example

Based on `http://pubs.opengroup.org/onlinepubs/009695399/functions/pthread_sigmask.html`
```
static sigset_t   signal_mask;  /* signals to block         */

int main (int argc, char *argv[])
{
    pthread_t  sig_thr_id;      /* signal handler thread ID */
    sigemptyset (&signal_mask);
    sigaddset (&signal_mask, SIGINT);
    sigaddset (&signal_mask, SIGTERM);
    pthread_sigmask (SIG_BLOCK, &signal_mask, NULL);

    pthread_create (&sig_thr_id, NULL, signal_thread, NULL);

    /* APPLICATION CODE */
    ...
}

void *signal_thread (void *arg)
{
    int       sig_caught;    /* signal caught       */

    sigwait (&signal_mask, &sig_caught);
    switch (sig_caught)
    {
    case SIGINT:     /* process SIGINT  */
        ...
        break;
    case SIGTERM:    /* process SIGTERM */
        ...
        break;
    default:         /* should normally not happen */
        fprintf (stderr, "\nUnexpected signal %d\n", sig_caught);
        break;
    }
}
```
(Under construction)