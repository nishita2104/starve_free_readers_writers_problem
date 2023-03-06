# Starve_Free_Readers_Writers_Problem
## Description:
Readers-Writers problem is a classical problem involving sharing of a memory or database by multiple processes concurrently. Readers read data from a shared location and writers write data to that shared location. Any number of readers can access the critical section simultaneously but only one writer can access the critical section at one point of time. While accessing the critical section, there could be synchronization problems. Thus, semaphores are used to solve those problems. However, using semphores could also lead to starvation of readers or writers based on their priority. Below is the starve free solution to this classical problem. 
## Data Structures used:
Queue (First-in-First-out structure): This is used to queue the processes that are blocked by the semaphore during the execution. 
## Semaphores:
Semaphore is a structure that is used to control the entry of processes in the critical section. It has two functions: `wait()` and `signal()`.
* `wait()`: It decrements the value of semaphore and pushes the particular process to the rear of the queue, thus blocking and removing that process from ready queue. 
* `signal()`: It increments the value of semaphore and if there are any processes in the blocked queue, then it pops the first process from the front of the queue and queues it in the ready queue. 
## Classical solution:
There is a classical solution to the readers-writers problem but it leads to the writers or the readers starving.
Following is the solution to the first readers-writers problem where there is a possibilty that the writers may starve. 
Both readers and writers share the same shared location/database.
We have a readcount variable (initially 0) and two semaphores: 
```cpp
mutex = 1 //mutual exclusion semaphore for updating readcount variable (initially 1)
wrt = 1 //mutual exclusion semaphore for writers (initially 1)
```

**Readers' Process:**
```cpp
do{
  wait(mutex);
  readcount++;
  if(readcount==1) wait(wrt);
  signal(mutex);
  /*

  Critical Section

  */
  wait(mutex);
  readcount--;
  if(readcount==0) signal(wrt);
  signal(mutex);
  
}while(true);
```

**Writers' Process:**
```cpp
do{
  wait(wrt);
  /*

  Critical Section

  */
  signal(wrt);
}while(true);
```

Here, as soon as a reader starts performing read operation, it makes the writer process to wait for `wrt` semaphore which is signalled by the reader process only when all readers are done performing their operations and all other readers wait on `mutex` semaphore. This could lead to starvation as all writers are paused on `wrt` till all the readers are done.

## Starve Free solution:
The solution above led to starvation as the readers could continuously enter the critical section and eventually block the writers. So the starve free solution is to add another semaphore `new_mutex` that could control which process enters the workflow of the above solution. This ensures that the readers coming in after the writers are pushed into the FIFO queue of the `new_mutex` rather than blocking the writers' access, thus making the algorithm starve-free.
Initially, semaphores:
```cpp
mutex = 1
wrt = 1
new_mutex = 1
readcount = 0

```
* Readers' Process:
```cpp
do{
  wait(new_mutex);
  wait(mutex);
  readcount++;
  if(readcount==1) wait(wrt);
  signal(mutex);
  signal(new_mutex);
  /*

  Critical Section

  */
  wait(mutex);
  readcount--;
  if(readcount==0) signal(wrt);
  signal(mutex);
}while(true);
```

* Writers' Process:
```cpp
 do{
  wait(new_mutex);
  wait(wrt);
  /*

  Critical Section

  */
  signal(wrt);
  signal(new_mutex);
}while(true);
```
Initially in the readers' process, the `wait()` function is called for `new_mutex`. This ensures that if writer process is going on, then the reader is pushed into FIFO queue of the `new_mutex` with other writer processes waiting. Thus, this enables both readers and writers to be of same priority and escape starvation.
In the writers' process, `wait()` function is called for `new_mutex` and is thus queued if a reader process is going on. 

This entire solution helps achieve the no starvation goal using a FIFO queue that implements a First-come-first-serve basis.  
