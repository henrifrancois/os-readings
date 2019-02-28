# Reading 01: Processes

## Chapter 6: Limited Direct Execution

### Time sharing is the method by which virtualization of the CPU is achieved, but it introduces several complications
  1. *Performance*
  	   Summary: How can virtualization be implemented without excessive overhead to the system?

   Potential solution #1: **_Limited direct execution_**
     
  2. *Control*
  	  Summary: How can we run those processes efficiently while maintaining control over the CPU? (i.e. how do we
           monitor, coordinate and modify processes)

      Potential solution #1: ...

### Performance enhancing solution #1: Limited Direct Execution
  when processes are run directly on the CPU.

	Several problems arise from limited direct execution, which are mainly control issues. First, there needs to exist some to limit the processe's ability to behave outside of what we deem acceptable. Second, how do we virtualize the CPU through time sharing if the CPU is dedicated to a single process?

**Ensuring Restricted Operations**
	_Cooperative Approach_
	Having two distinct modes of operation can ensure restricted operations for appropriate users to some degree. This is done in Unix systems by designing two modes: user mode and supervisor (or kernel) mode. supervisor mode enables the os to perform restricted operations such as different usage modes restrict the user, but there are times when it is necessary for a user process to make use of kernel resources or features. This need brings justifies the existence of **system calls** which are used as a means of interfacing between the process and the kernel. To make use of a kernel resource from the user mode, system calls use a trap instruction which saves the state of the user process on the kernel stack before going into the system call, then after performing their duty, returns to the initial process through the use of a return-from-trap instruction (This instruction restores the state of the innitial calling process by popping all registers from the kernel stack pointer).
	_Non-Cooperative Approach_
	Devices can be programmed to raise interrupts at regular intervals. Once these interrupts are raised the OS can take over and halt the process, and a interrupt-handler is run. The OS can switch to another process if need be at this point.
	Configuration of the hardware interrupts is done at boot time. 

**Important definitions**
limited direct execution: two phase protocol during which
	process runs directly on the CPU. 
	In the first stage,the OS initializes the trap table at boot time.
	In the second, a running process traps into the OS
	via a system call, then goes back to user mode via 
	the return from trap instruction.
	(the OS also uses the return-from-trap to begin
	running the process)
user mode: limited privileges and access to
	system resources
kernel mode: complete access to system resources
system call: framework or api for utilizing 
	system resources
trap: instruction which stops the running process, copies the
	state of the process to the kernel stack, and enters
	kernel mode for accessing system resources in a controlled fashion.
trap table: kernel data structure which contains predefined 
	trap handlers which determine the expected behavior hardware
	in case of a trap function.
return-from-trap: instruction which returns from trap
	_safely_
context-switch: defined as the changes from the kernel
	mode to the user mode, during which the state of
	the process is transferred over to the kernel stack
	(or to the process' stack at the return-from-trap)
	by pushing the general purpose registers to the 
	kernel/process stack. 
timer interrupt: hardware programmed / defined interrupt
	which enables the OS to handle a proces in a non-
	cooperative manner


## Chapter 7: CPU Scheduling

**Assumptions made:**
  1. equal length jobs
  2. identical arrival times
  3. jobs allways complete
  4. Jobs only use the cpu
  5. knowledge of job run time

### First In First Out (FIFO)
Advantages:
  * Simple implementation

Shortcomings:
  * short jobs at the end of the queue are held back by longer jobs
(remove assumption 1)

### Shortest Job First (SJF)
Advantages:
  * Short jobs are handled first
  * Shorter turnaround time compared to FIFO

Shortcomings:
  * Smaller jobs arriving after longer jobs are handled later
  implying the same convoy problem
(remove assumption 2)

### Shortest Time-to-Completion First (STCF)
Advantages:

  * Late short jobs are not held back in the queue in favor of longer jobs

Shortcomings:
  * Bad response times

  * Cannot account for I/O operations


### Round Robin (time slicing)
Description: Each job is run for a time slice (shceduling quantum) and switches to the next job in the queue.
Scheduler runs each slice until job completes, for each job

Advantages:
  * Excellent response time
  * If slice size is well defined, cost of job switching is amortized

Shortcomings:
  * Terrible turnaround time

### I/O operation scheduling
I/O operations can be scheduled in the following way:
Treating each chunk of a job performing I/O operations as an independent job, the scheduler can then fill in the 
gaps between I/O operations (when the CPU is not being used) to schedule other jobs

**Important Definitons**
workload: collective assumptions about the processes 
	running in the system
turnaround time: job completion time - job arrival time
response time: job scheduling time - job arrival time
fairness: the propensity for a scheduler to schedule all jobs
amortization: reduction of cost of an operation by reducing
	the frequency of that operation
overlap: 

## Chapter 8: Multi-Level Feedback Queue (MLFQ)
Goals: 
  * optimize turnaround time 
  * attain some responsiveness

Assumptions: None

MLFQ is comprised of distinct queues, each assigned a certain
priority level.

### Basic functionality
Rule 1: Priority(A) > Priority(B) => A runs, B doesn't.
Rule 2: Priority(A) = Priority(B) => A & B run utilizing Round Robin

### How are priorities determined? 
	Job priority is varied over time based on a job's observed behavior

### How do priorities change? 
Rule 3: Arriving jobs are placed in the hisghest priority queue
Rule 4a: If a job uses its entire time slice when running, then its priority is reduced
Rule 4b: If a job gives up the CPU before its time slice expires, its priority is maintained (because it likely uses many I/O operations)
time slices are predefined for each queue
#### Shortcomings so far:
  * I/O intensive tasks can monopolize the CPU -> long running jobs never get CPU time
  * Scheduler is gameable by well timed I/O calls.

### Priority Boosting
Rule 5: After some time _S_, move all jobs in the system to the topmost queue

Advantages:
  * Processes all receive CPU time
  * Changing behavior can be captured and adapted to to some degree

### Better Accounting
Problem: Scheduler gaming is still possible by giving up the CPU at the right time.

Replacing rules 4a and 4b with rule 4.

Rule 4: Once a job uses up its time allotment at a given level (regardless of how many times it has given up the CPU), its priority is reduced (i.e., it moves down one queue).

### Last Issues
How to parametrize the Scheduler?
_Ousterhout's Law_: Avoid constants for which good values are impossible to determined without black magic.
					Set defaults, which can be modified to improve performance at later times.


## Chapter 9: Proportional Share Scheduler (Fair-share scheduler)

### Lottery Scheduling
In LS, tickets represent the total percentage of system resources a job should receive.
To determine a job's ticket count, a lottery is held at regular intervals. (some n time slices)
Winning tickets determine a job's resources. Because of randomness, accurate ticket allocation is
more likely with longer running processes than with shorter ones. 

Ticket currencies allow a user to allocate any number of tickets to processes, which are then converted
to the correct number of 'system' tickets. This allows different users with a different number of jobs
to allocate resources which the sytem can then appropriately schedule.

Advantages:
  * Achieves proportional time sharing through randomness
  * Process time-share scaling is possible through the use of ticket currencies
  * no global state

Shortcomings:
  * No way of telling how to allocate tickets to jobs

### Stride Scheduling

Advantages:
  * Deterministic allocation of tickets for system resource utilization

Shortcomings:
  * New jobs can monopolize the CPU if the correct pass value is not set

### Completely Fair Scheduler
Goals: fairly divide CPU time among all processes

Processes accumulate `vruntime` as they run, and CFS picks the job with
the lowest `vruntime`. CFS manages control of the scheduling through 
control parameters. 

sched_latency: designated `vruntime` for a process. This value is 
	divided by the number of processes running to determine slice
	width per process. 
min_granularity: minimum slice width

CFS makes use a Red-Black tree to keep track of the different running and/
or runnable jobs. This, in part, is what makes CFS efficient and scalable. 

#### I/O and sleeping processes
The `vruntime` of a sleeping job is altered once it starts running again.

Advantages:
  * Scalable fair-share scheduling


**Important Definitions**
ticket: share of a system's resources
ticket currency: user defined ticket
ticket transfer: action of transferring tickets between users
ticket inflation: temporary increase in the number of tickets of a user
stride: value inversely proportional to the number of tickets which helps
	the scheduler determine which process to run.
pass: counter being incremented with the stride, for each process.
	the process with the lowest pass is always run first by the OS. 

## Chapter 10: Advanced Multiprocessor Scheduling
Problem: How do you schedule jobs when dealing with multiple processors?

Main issue lies in the fact that a process won't always be run on the same 
CPU, meaning its state will be shared among multiple caches. The caching
issues of coherency, affinity and synchronization also play a factor.


### Single Queue Scheduler
All jobs in a single queue

Advantages:
  * Simple implementation

Shortcomings:
  * Not scalable due to locking


### Multi Queue Scheduler
Each CPU has its own scheduling queue. The scheduling on each queue is then
handled by some scheduling scheme like Round Robin or CFS

Since one queue can have way more jobs scheduled than another, queues transfer
jobs between them through the _**migration**_ process. This is called **load balancing**.

Advantages:
  * Scalable
  * Good Cache Affinity

Shortcomings:
  * Susceptible to Load Imbalance
  * Complex Implementation

### Other Schedulers
O(1) & 
	Multiple queued approach. Priority based. 

CFS (Completely Fair Scheduler)
	Multiple queued approach. Deterministic proportional-share scheduler. 

BF (Best Friend Scheduler)
	Single queued approach. Proportional-share, Earliest Eligible Virtual Deadline First approach.

# Question:
Can a ticket be faked by a process in Lottery Scheduling?
