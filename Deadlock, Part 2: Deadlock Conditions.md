## Coffman conditions
There are four _necessary_ and _sufficient_ conditions for deadlock. These are known as the Coffman conditions.

* Mutual Exclusion
* Circular Wait
* Hold and Wait
* No pre-emption

All of these conditions are required for deadlock, so let's discuss each one in turn. First the easy ones-
* Mutual Exclusion: The resource cannot be shared
* Circular Wait: There exists a cycle in the Resource Allocation Graph. There exists a set of processes {P1,P2,...} such that P1 is waiting for resources held by P2, which is waiting for P3,..., which is waiting for P1.
* No pre-emption: Once a process has acquired a resource, the resource cannot be taken away from a process and the process will not voluntarily give up a resource.
* Hold and Wait: A process acquires an incomplete set of resources and holds onto them while waiting for the other resources.

## Examples

Two students need a pen and paper:
* The students share a pen and paper. Deadlock is avoided because Mutual Exclusion was not required.
* The students grab both the pen and paper in one operation ("Get both or get none"). Deadlock is avoided because there is no _Hold and Wait_
* The students both agree to grab the pen before grabbing the paper. Deadlock is avoided because there cannot be a circular wait.
* The students are friends and will ask the each other to give up a held resource. Deadlock is avoided because pre-emption is allowed.


## Livelock
Live is _not_ deadlock-

Consider the following 'solution'
* The students will put down one hold resource if they are unable to pick up the other resource within 10 seconds. This solution avoids deadlock however it may suffer from livelock.

Under construction...





