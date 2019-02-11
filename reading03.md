# Chapter 30: Condition variables

## Definition and Routines
At times, it is necessary to plan execution of some code in a thread based on a condition being true in another thread. While it is possible and even easy to simply halt a thread form proceeding until a condition in another thread becomes true, it wastes CPU cycles and is inefficient. _There must be a better way!_

A **condition variable** is used in place of a spind-and-wait approach to conditional execution within the context of multithreaded work.

The condition variable itself is a queue which threads can put themselves in to wait for a change in a predefined condition. The thread in which the condition is set to change may then wake up one or more thread(s) which were awaiting for a change of state in the condition variable. This is done by the concerned thread sending a 'wake up' signal to the other thread(s).

**K.I.S. rules to avoid chaos when calling `wait()` and `signal()`**
> hold the lock when calling `wait()`
> hold the lock when calling `signal()`

## The Bounded Buffer Problem
Considering two distinct groups of threads:
1. *Producer threads*, which generate data items and put them in a buffer
2. *Consumer threads*, which take data items from the buffer, and make use of them
Examples: multi-threaded http servers; unix pipes

This pattern creates a potential for concurrency problems because both the producers and consumers are making use of a shared resource: the buffer. There isn't a guarantee that the producers and consumers are synchronized in some way and this can cause terrible terrible consequences , therefore it falls to us to create those guarantees. 
First, we can decouple the two sets of actions taken by the consumers and producers by checking for a certain condition in order for those actions to be taken (consumers do their thing only if a == 0 & producers do their thing only if a == 1). This is a first step, but is not sufficient. Additional conditions must be met to ensure that a producer only writes to a buffer when that buffer has space available, and a consumer only gets data from a buffer if that buffer has data in the first place.
Additional consumers and producers complicate our task. Data races are possible even _among_ the producers or consumers if they do not have a strict verification and signaling process some of the conditions stated above. For instance, once a consumer has consumed a piece of data from the buffer, and the buffer is empty, it has to signal the correct thread (a producer) which acts in a complementary way. 

## Covering Conditions
At times, the most appropriate thread to wake is difficult to distinguish automatically, and a call to `signal()` will be an ambiguous call to wake up a thread. Then, a possible solution is to wake up all threads: threads for which the condition remain false will go back to sleep, while threads which can continue on will remain awake.

# Chapter 31: Semaphores
A semaphore is an object which can be manipulated through two routines: `sem_wait()` and `sem_post()`.
As with everything in this section, semaphores must be properly initialized before use.

Semaphores can be interacted with through two the two interface functions/methods mentioned previously and they work as follows:
+ `sem_wait()` either immediately returns or suspends execution in preparation for a `sem_post()`.
+ `sem_post()` will increment the value of the semaphore and if there is a thread to be woken up, wakes it up.
+ The value of a semaphore, if negative, reflects the number of queued threads. 

## Semaphores as Locks
Wapping a critical section in a sem_wait()/sem_post() pair results in a mutex equivalent. It suffices to initialize the semaphore to 1, so that the first thread can directly run to completion while any other thread has to wait for that first thread to complete because their call to sem_wait() results in the semaphore's value to be negative. Such a semaphore is referred to as being a **nary semaphore**.

API
```c
#include <semaphore.h>

sem_t s;
sem_init(&s, 0, 1);

int main(){
sem_wait(&m);
...
sem_post(&m)
}
```
## Semaphores for Ordering
If a thread must wait for another thread before completing its execution, a semaphore can help enforce such an ordering. By calling `sem_wait()` within the parent and `sem_post()` within the child, a semaphore can be used to stall the parent thread in anticipation of the child thread's exit. 

## Bounded Buffer Problem (remix)
As we've seen in the previous chapter, synchronization between producer and consumer workflows is essential to maintaining sanity within the buffer, which is a critical section. _**Binary semaphores**_ can be used to fill and consume data from the buffer without data races, but caution must be exerced when acquiring and releasing locks to ensure no deadlocks occur.

## Reader-writer Locks
As different types of operations can be quite different, the effect of concurrency can cause very different needs in terms of locking. For instance, as write operations within a linked list can cause much more chaos than any read operation can hope to, locking for reads can be made much more flexible and permissive than for the write operations. **Reader-writer locks** were built for such situations. 
Such locks allow multiple readers to access the list at the same time (multiple readers can acquire the read lock, while no writers are able to acquire the write lock at that time, which makes sense but can starve writer locks). Furthermore, only a single write lock can be acquired at a time. 

## Dining Philosopher Problem
This is a problem about deadlock. If all process need to acquire multiple resources that are shared between them. Acquiring a resource needed by another is a sure way to reach deadlock. One solution to this problem is to create different resource acquisition patterns for the threads at play. If a thread can acquire a resource which will make it easier for another thread to acquire another resource further down the line, then it can ease the deadlock and increase concurrency. For instance, making it so that one thread can't acquire a resource by making another thread acquire it first, then another thread may gain access to another resource it wouldn't be able to acquire otherwise.

## Zemaphores

API
```c
typedef struct __Zem_t {
 int value;
 pthread_cond_t cond;
 pthread_mutex_t lock;
} Zem_t;

// only one thread can call this
void Zem_init(Zem_t *s, int value) {
 s->value = value;
 Cond_init(&s->cond);
 Mutex_init(&s->lock);
}

void Zem_wait(Zem_t *s) {
  Mutex_lock(&s->lock);
  while (s->value <= 0)
    Cond_wait(&s->cond, &s->lock);
  s->value--;
  Mutex_unlock(&s->lock);
 }

void Zem_post(Zem_t *s) {
 Mutex_lock(&s->lock);
 s->value++;
 Cond_signal(&s->cond);
 Mutex_unlock(&s->lock);
}
```


# Chapter 32: Concurrency Bugs

## Non-Deadlock Bugs
Although deadlock is one of the most infamous concurrency bugs, the majority of concurrency issues aren't deadlock related. Non-deadlock concurrency issues can be categorized as:
+ atomicity violation bugs
  Atomicity violation bugs, as defined by Lu et al. occur when a code region is intended to be atomic but the atomicity is not enforced during execution.
  For instance, this section of code:
  ```c
  1 Thread 1::
  2 if (thd->proc_info) {
  3 ...
  4 fputs(thd->proc_info, ...);
  5 ...
  6 }
  7
  8 Thread 2::
  9 thd->proc_info = NULL;
  ```
  has one thread setting a null value where another is avoiding one. With some unfortunate interruption timing, this is a hotbed for problems. This can be remediated by adding locks to the changing variable and to the vulnerable section calling `fputs`:
  ```c
  1 pthread_mutex_t proc_info_lock = PTHREAD_MUTEX_INITIALIZER;
  2
  3 Thread 1::
  4 pthread_mutex_lock(&proc_info_lock);
  5 if (thd->proc_info) {
  6 ...
  7 fputs(thd->proc_info, ...);
  8 ...
  9 }
  10 pthread_mutex_unlock(&proc_info_lock);
  11
  12 Thread 2::
  13 pthread_mutex_lock(&proc_info_lock);
  14 thd->proc_info = NULL;
  15 pthread_mutex_unlock(&proc_info_lock);
  ```
+ order violation bugs
  These types of bugs occur when the desired order between two different groups of memory accesses is flipped. This might mean that some sequence of events results in a state being reached or a function being called without the necessary prior steps having been completed. 
  This can be mitigated by the use of condition variables, which as we've seen before, help in the correct handling of sequences of events involving multiple threads.
  ```c
  1 pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
  2 pthread_cond_t mtCond = PTHREAD_COND_INITIALIZER;
  3 int mtInit = 0;
  4
  5 Thread 1::
  6 void init() {
  7 ...
  8 mThread = PR_CreateThread(mMain, ...);
  9
  10 // signal that the thread has been created...
  11 pthread_mutex_lock(&mtLock);
  12 mtInit = 1;
  13 pthread_cond_signal(&mtCond);
  14 pthread_mutex_unlock(&mtLock);
  15 ...
  16 }
  17
  18 Thread 2::
  19 void mMain(...) {
  20 ...
  21 // wait for the thread to be initialized...
  22 pthread_mutex_lock(&mtLock);
  23 while (mtInit == 0)
  24 pthread_cond_wait(&mtCond, &mtLock);
  25 pthread_mutex_unlock(&mtLock);
  26
  27 mState = mThread->State;
  28 ...
  29 }
  ```
  
## Deadlock bugs
Four key conditions must be met in order for a deadlock to occur:
1. **Mutual Exclusion** must be in use: a thread needs exclusive access to a resource it requires
2. **Hold-and-Wait**: threads must be awaiting a free lock while simultaneously holding another lock.
3. **No Preemption**: resources cannot be forcibly removed from threads that are holding them.
4. **Circular wait**: there exists multiple threads where each threads hold a lock which another thread is waiting to be freed, in a such a manner that threads are waiting for resources in a circular way.

### Prevention
Locking schemes can be written to avoid circular wait situations. One way of achieving this is by consistently acquire a certain lock before another. This is known as **total lock ordering**. In more complex systems where detailed orderings for locks is hard to achieve, a **partial ordering** is a more feasible way of ordering certain groups of types of locks instead of individual ones.

### No Preemption
Trying to grab a lock and in case of failure, find something else to do and try later. Don't stay idle while waiting for a lock to be freed. 

### Mutual Exclusion
If certain operations can be achieved atomically, the use of muteces is unnecessary and we can prevent deadlocks.

### Deadlock Avoidance via Scheduling
A scheduler could use knowledge of the different threads' behaviors to avoid deadlocks. For instance, if threads A and B are prone to deadlock since they acquire the same locks, the scheduler could make sure that A & B are never 'running' in such a way that a deadlock could occur.

### Detect and Recover
If deadlocks are very rare, just let the few that do occur happen, and move on.
