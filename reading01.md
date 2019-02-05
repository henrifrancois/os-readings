# Reading 01: Processes

## Chapter 6: Limited Direct Execution

Still to be discussed:
+ Trap table
+ Trap table setup
+ Timer interrupt
+ Context and context switching



  ### Time sharing is the method by which virtualization of the CPU is achieved, but it introduces several complications

      1. *Performance*
      	   Summary: How can virtualization be implemented without excessive overhead to the system?

	   Potential solution #1: *_Limited direct execution_*
	     
      2. *Control*
      	  Summary: How can we run those processes efficiently while maintaining control over the CPU? (i.e. how do we
	           monitor, coordinate and modify processes)

          Potential solution #1: ...

  ### Performance enhancing solution #1: Limited Direct Execution

      Definition: processes are run directly on the CPU.

      Several problems arise from limited direct execution, which are mainly control issues. First, there needs to exist
      some to limit the processe's ability to behave outside of what we deem acceptable. Second, how do we virtualize
      the CPU through time sharing if the CPU is dedicated to a single process?

      **Ensuring Restricted Operations**
	Having two distinct modes of operation can ensure restricted operations for appropriate users to some degree.
	This is done in Unix systems by designing two modes: user mode and supervisor (or kernel) mode.
	supervisor mode enables the os to perform restricted operations such as 
	  Different usage modes restrict the user, but there are times when it is necessary for a user process to
	make use of kernel resources or features. This need brings justifies the existence of **system calls** which
	are used as a means of interfacing between the process and the kernel. To make use of a kernel resource from
	the user mode, system calls use a trap instruction which saves the state of the user process on the kernel
	stack before going into the system call, then after performing their duty, returns to the initial process through
	the use of a return-from-trap instruction (This instruction restores the state of the innitial calling process by popping all registers from the kernel stack pointer).


#### Questions
+  Direct execution seems to imply that there are other ways of running a program besides running it on the CPU

### *Main Question*



## Chapter 7: CPU Scheduling

Still to be discussed:


## Chapter 8: Multi-Level Feedback Queue

## Chapter 9: Scheduling Proportional Share

## Chapter 10: Advanced Multiprocessor Scheduling