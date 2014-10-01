## What is the Reader Writer Problem?

Imagine you had a key-value map data structure which is used by many threads. Multiple threads should be able to look up (read) values at the same time provided the data structure is not being written to. The writers are not so gregarious - to avoid data corruption, only one thread at a time may modify (`write`) the data structure (and no readers may be reading at that time). 

The is an example of the _Reader Writer Problem_  Namely how can we efficiently synchronize multiple readers and writers such that multiple readers can read together but a writer gets exclusive access?

An incorrect attempt is shown below (lock is a simple pthread_mutex_lock):

<table><tr><td>
<pre>read()
  lock(m)
  // do read stuff
  unlock(m)
</pre>
</td><td>
<pre>write
  lock(m)
  // do read stuff
  unlock(m)
</pre></td></tr></table>

At least our first attempt does not suffer from data corruption (readers must wait while a writer is writing and vice versa). However readers must also wait for other readers. So let's try another implementation..

Attempt #2:
<table><tr><td>
<pre>read()
  while(writing) {/*spin*/}
  reading = 1;
  // do read stuff
  reading = 0;
</pre>
</td><td>
<pre>write
  while(reading || writing) {/*spin*/}
  writing = 1;
  // do write stuff
  writing = 0;
</pre></td></tr></table>


Under construction