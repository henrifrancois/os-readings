# Reading 02: Threads

## Chapter 26: Concurrency and Threads

**Similarities between threads and processes**

_Context Switch_
Changing from one thread to another, just like for processes, requires to CPU to make a context-switch

| | Processes | Threads 
|---|---|---|
|Saving state | PCB | TCB |
| Address Space | Unique | Shared |


**Why use threads at all?**

1. Parallelization
   Multi-threaded programs can take advantage of multiple cores to speed up their execution.

2. Avoiding I/O blocking
   Similarly to how **overlap** in scheduling enabled processes to circumvent wait times enforced by I/O calls, Can be seen as multi programming, but within a program.

While multiple processes could, in theory, be used to do several operations individually, or even do the work one process in multiple steps to avoid I/O blocking issues, threads share a single address space, making it easier to share data than would be the case for different processes. 

Threads do not necessarily run in the order of their creation. Just like processes, threads are handled by the scheduler, which offers no guarantees on process - or thread - running order. 

**Important Definitions**

> multi-threaded program: Program made up of distinct, executable chunks called threads.

> Thread: independent sequence of executions running in a shared address space.

> Parallelism: 

> Concurrency:

> race condition: flaw which occurs when the timing or ordering of events in a program affect a program's correctness.

> data race: Situation resulting from two memory accesses in a program where both target the same location and are performed by two threads

> critical section: section of code that accesses a shared variable and must not be currently executed by more than one thread, because it is susceptible of raising a race condition.

> synchronization primitives: tools (instructions) used to ensure deterministic outcomes when dealing with critical sections being accessed by multiple threads.

> mutual exclusion: Guarantee that multiple threads will never access a critical section concurrently. 


## Chapter 27: Thread API

### Thread Creation

Using the thread creation api's `pthread_create()` function enables the creation of threads.

```c
#include <pthread.h>

int pthread_create(
    pthread_t *			thread,
    const pthread_attr_t * 	attr,
    void *		 	(*start_routine) (void*),
    void * 			arg;
) 
```

### Thread Completion

Awaiting the completion of a thread requires the use of the `pthread_join()` function.

```c
#include <pthread.h>

typedef struct __myarg_t {
   int a;
   int b;
} myarg_t;

void *mythread(void *arg) {
   myarg_t *m = (myarg_t *) arg;
   printf("%d %d\n", m->a, m->b);
   return NULL;
}

int main(int argc, char *argv[]) {
   pthread_t p;
   int rc;

   myarg_t args;
   args.a = 10;
   args.b = 20;
   rc = pthread_create(&p, NULL, mythread, &args);
   ...
}
```

### Locks

Locks are synchronization primitives which help secure and isolate critical sections to ensure correct operation.
Essentially, a lock permits the locked thread to be accessed by another only after the locked thread has run to completion.
Locks, like all data structures in C, must be properly initialized. 

```c
int pthread_mutex_lock(pthread_mutex_t *mutex)
int pthread_mutex_unlock(pthread_mutex_t *mutex)
```

The lock requires initialization, which can be done in one of two ways:

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```
OR
```c
int rc = pthread_mutex_init(&lock, NULL);
assert(rc == 0);
```

### Condition variables
Condition variables are used to enable communication (to some degree) between different threads.
Passing a signal from one thread to another enables some (inter)dependent behavior between threads A & B.

```c
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex)
int pthread_cond_signal(pthread_cond_t *cond)


pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

Pthread_mutex_lock(&lock);
while (ready == 0){
      Pthread_cond_wait(&cond, &lock);
}
Pthread_mutex_unlock(&lock);
```

Signaling requires locks to be held prior.

**THREAD API GUIDELINES**

+ Locking and signaling between threads should be kept as simple as possible.

+ Interactions between threads should be kept at a minimum because more complexity -> higher risk of race conditions.

+ Locks and condition vars should be initialized.

+ Each thread has its own stack.

+ If you're passing a reference to a stack allocated value, you're doing it wrong.

+ Always use condition vars to signal between threads.


## Chapter 28: Locks
At its core, a lock is a variable belonging to a thread. This lock can either be available (free, unlocked) or unavailable (held, locked). At any time, only a single thread holds a lock and is in a specific critical section.

### Workflow:
+ Thread acquires the lock(). If it is held by another thread, then it is (presumably) queued to wait for the lock to be freed. 

+ Once the thread releases the lock, another thread waiting for the lock is notified and takes ownership of the lock.

### Building, Evaluating
Efficient Locks provide mutual exclusion at low cost. How do we build and evaluate such a lock.

Evaluating the efficacy of a lock relies on some key factors:
+ Mutual Exclusion: does the lock achieve its basic function in all conditions. 

+ Fairness: Is there a tendency for threads awaiting locks to be freed to starve

+ Performance: How much slower does the addition of this lock make the completion of this task. This assessment is reliant on multiple factors:
  1. No contention cases
  2. Multiple contenders, single CPU
  3. Multiple contenders, multiple CPUs

### Controlling Interrupts
Single processor systems disabled interrupts during a thread operation to achieve atomic executions on critical sections. If trap instructions aren't disrupting the execution of a critical section, then the operation either completes or fails, which ensures thread safety. This approach is ruined by the advent of multi-core CPUs. Additionally, this requires any thread making use of such a system to execute a privileged operation. Lastly, this can result in missing important information because interrupts are turned off. 

### Loads and Stores as Locks | Spin Locks
Two variables can be used to signal whether a thread holds a lock or not, which has the downsides being inefficient and incorrect. One variable signals that the lock has been acquired and the second specifies the owner from a number of threads. 


### Compare-and-Swap Locks
Keep track of the value of interest, and if the value on  read is equal to the expected value, then update the value. Otherwise, do nothing. 

### Load Links and Store Conditionals
A traditional load instruction, coupled with a store instruction which only updates register information if there were no conflicting stores.

### Fetch-and-Add
Increments register value while returning the old register value. This enables the creation of a ticket lock.

### Yield Locks
System locks such as the three mentioned above are inefficient at times. Yielding simply means to give up the CPU, by moving the thread from the running state to the ready state. We can think of this as self-descheduling. 

### Queue-based approach


### Two-phased Locks
Moves a waiting thread from the spinning phase to sleep if no lock is acquired promptly. 

**Important Definitions**
mutex: POSIX name for a lock since it provides _mutual exclusion_
coarse-grained locking: single lock for controlled access to **any** critical section
fine-grained locking: multiple locks which control access to _different_ critical sections. (This can increase concurrency)


## Chapter 29: Lock-based Concurrent Data Structures

The addition of locks to a data structure makes that data structure thread-safe. In implementing thread-safe data structures, there is constantly an optimization concern for the trade offs of high performance and concurrency.

### Concurrent Counters
As even keeping track of the changes to a value or updating a value across different threads can be a cause of data races, a thread-safe counter data structure can mitigate this issue.
A thread-safe counter can be implemented such that it contains a lock, which is acquired by any calling routine/thread, and released once it is returning from the call.

**Approximate Counter**
Containing one local counter per CPU and a global counter. Each local counter contains a lock, as does the global counter.
Thread running on a certain CPU increment their local counter, and the local counters synchronize their current state with the global
counter at regular intervals. The rate of synchronization is determined by some value S.
Lower S imply less scalable design, but higher fidelity between the actual values of the counters.
Higher S implies the opposite.

### Concurrent Linked Lists
A simple concurrent linked list implements a lock on the list for insertion, deletion and lookup operations. These operations can fail on memory allocation for each new node, although rather rare. The solution to this problem is avoiding or limiting the program/thread flows to those that minimize failure possibilities, but code of this type doesn't scale. Scaling a concurrent linked list can be done (in theory) with the use of a hand-over-hand locking mechanism, where instead of a single lock per linked list, every node in a linked list is equipped with a lock. As with anything in operating systems, we are left with trade offs, in this case in speed versus ability to scale? 

### Concurrent Queues
Concurrent queues do not make use of a single, global lock. Instead, concurrency is increased by adding a lock at the entry and exit of the queue, to control the flow of queuing and dequeuing operations. To distinguish between the entry and exit of the queue, it is initialized with a dummy node.

### Concurrent Hash Tables
Concurrent fixed size hash tables (with separate chaining) can be implemented using the concurrent linked lists. Good performance is achieved thanks to the fact that each bucket has a lock, enabling multiple operations on multiple threads until a collision occurs.

> "Premature Optimizations are the root of all evil." - Donald Knuth

**Important definitions**
Perfect Scaling: n threads complete as quickly on n processors as 1 thread completes on 1 processor.
hand-over-hand locking: enabling a lock a each node of a linked list allows for safer manipulations of data at a node level.

# Question
At thread creation time, could metadata be added to the thread to help with ordering? Would this be a problem to the scheduler? 
Is asynchronous programming the same as parallel programming? How are these two concepts different? Are there advantages to using one over the other?
How to identify critical sections?
There are a number of ways
