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


# Chapter 32: Concurrency Bugs
