## What is a Resource Allocation Graph?
A resource allocation graph tracks which resource is held by which process and which process is waiting for a resource of a particular type. It is very powerful and simple tool to illustrate how interacting  processes can deadlock.


If there is a cycle in the Resource Allocation Graph then the processes will deadlock. For example, if process 1 holds resource A, process 2 holds resource B and process 1 is waiting for B and process 2 is waiting for A, then process 1 and 2 process will be deadlocked.

Here's another example, that shows Processes A,B acquiring resources A,B while process C is waiting ro resource X-



![ResourceAllocationGraph-Ex1.png](https://raw.githubusercontent.com/wiki/angrave/SystemProgramming/ResourceAllocationGraph-Ex1.png)

In addition, if there are other processes waiting for resource A and/or resource B then they will also be deadlocked.


Todo: More complicated example