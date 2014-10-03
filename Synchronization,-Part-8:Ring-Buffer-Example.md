
# Under construction
## What is a ring buffer?
A ring buffer is a simple typically fixed-sized storage mechanism where contiguous memory is treated as if it is circular, and two index counters keep track of the current beginning and end of the queue. 

A simple (single-threaded) implementation is shown below. Note enqueue and dequeue do not guard against underflow or overflow - it's possible to add an item when when the queue is full and possible to remove an item when the queue is empty.

```C
void* buffer[16];
int in = 0, out = 0;

void enqueue(void* value) { /* Add one item to the front of the queue*/
  buffer[in] = value;
  in ++; /* Advance the index for next time */
  if(in == 16) in = 0; /* Wrap around! */
}

void* dequeue() { /* Remove one item to the end of the queue*/
  void* result = buffer[out];
  out ++;
  if(out == 16) out = 0;
}
```

## What are gotchas of implementing a Ring Buffer?
It's very tempting to write the enqueue or dequeue method in the following compact form (N is the capacity of the buffer e.g. 16):
```C
void enqueue(void* value)
  buffer[ (in++) % N ] = value;
}
```
This method would appear to work (pass simple tests etc) but contains a subtle bug. With enough enqueue operations (a bit more than two billion) the int value of `in` will overflow and become negative! The modulo (or 'remainder') operator `%` preserves the sign. Thus you might end up writing into `buffer[-14]`  for example! 

A compact form is correct uses bit masking provided N is 2^x (16,32,64,...)
```C
buffer[ (in++) & (N-1) ] = value;
```

This buffer does not yet prevent buffer underflow or overflow. For that, we'll turn to our multi-threaded attempt that will block a thread until there is space or there is at least one item to remove.

## Checking a multi-threaded implementation for correctness

The following code is an incorrect implementation. What will happen? Will `enqueue` and/or `dequeue` block? Is mutual exclusion satisfied? Can the buffer underflow? Can the buffer overflow?

<table><tr><th>Initialization and global vars</th><th>enqueue</th><th>dequeue</th></tr>
<tr><td><pre>
p_m_t lock
sem_t s1,s2
p_m_init(&lock,NULL)
sem_init(&s1,0,16)
sem_init(&s2,0,0)
// + above code from #1	
</pre></td><td>

<pre>enqueue(void*value){

 p_m_lock(&lock)
 sem_wait( &s1 )
 
 buffer[ (in++) & (N-1) ] = value;

 sem_post(&s1)
 p_m_unlock(&lock)
</pre>
</td>
<td>
<pre>void* dequeue(){
  p_m_lock(&lock)
  sem_wdait(&s2)
  void * result = buffer[(out++) & 15 ]
  sem_post(&s2)
  p_m_unlock(&lock)
  return resul;
}</pre>
</td>
</table>
