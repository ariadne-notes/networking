#### IO Pathways

Device controller tells the CPU it's done (put data into a buffer) by sending an interrupt.

IO goes from controller - local buffer - CPU

#### Interrupts

Hardware interrupts
* A buffer has been filled

Traps or exceptions are software generated interrupts
* User requests
* Errors

Most operating systems are interrupt driven.

# Storage Structures

### Main Memory (DRAM)
* Random Access
* Lost with power outage (volatile)

### Secondary Storage
* Larger
* Not lost with power outage (non-volatile)

### Caching
Copying data from secondary storage to main memory
* Faster

Storage Hierarchy
Registers > cache > main memory (dram) > solid-state disks > spinning disks > optical disks > magnetic tapes.

### Direct Memory Access (DMA)
Some amount of DRAM is owned directly by an IO controller, and uses the DRAM for the buffer.
When done, the IO controller sends an interrupt.

# Processing
* **Asymmetric** - each processor does a specific task.
* **Symmetric** - each processor performs all tasks.

## Multithreading 
While one thread is asking for memory, execute the other thread. Go back and forth.


## Dual Mode
User mode and Kernel mode, with a mode bit.
Kernel mode is also called privileged.

#### System Calls
System calls are how user mode apps interact with the kernel.
APIs are provided facilities to access the kernel without using system calls (which may not be allowed)
* Win32 for Windows
* POSIX API (Unix, Linux, Mac OS X)
* Java API for Java Virtual Machine (JVM)

## Load Averages
Windows will show a percentage of CPU. Linux systems instead show the number of processes waiting to acces the CPU. It can get to double digits.

## Threading
A single-thread process has a program counter that says "go here to read the next instruction please"

## Memory Management
Copying from storage into dram, into cache. Only stuff in L1 cache can be executed.

```
           0.5 ns - CPU L1 dCACHE reference
           1   ns - speed-of-light (a photon) travel a 1 ft (30.5cm) distance
           5   ns - CPU L1 iCACHE Branch mispredict
           7   ns - CPU L2  CACHE reference
          71   ns - CPU cross-QPI/NUMA best  case on XEON E5-46*
         100   ns - MUTEX lock/unlock
         100   ns - own DDR MEMORY reference
         135   ns - CPU cross-QPI/NUMA best  case on XEON E7-*
         202   ns - CPU cross-QPI/NUMA worst case on XEON E7-*
         325   ns - CPU cross-QPI/NUMA worst case on XEON E5-46*
      10,000   ns - Compress 1K bytes with Zippy PROCESS
      20,000   ns - Send 2K bytes over 1 Gbps NETWORK
     250,000   ns - Read 1 MB sequentially from MEMORY
     500,000   ns - Round trip within a same DataCenter
  10,000,000   ns - DISK seek
  10,000,000   ns - Read 1 MB sequentially from NETWORK
  30,000,000   ns - Read 1 MB sequentially from DISK
 150,000,000   ns - Send a NETWORK packet CA -> Netherlands
|   |   |   |
|   |   | ns|
|   | us|
| ms|
```

Source [Stack Overflow](https://stackoverflow.com/questions/4087280/approximate-cost-to-access-various-caches-and-main-memory)

# Debugging

Kernighan's Law

> Everyone knows that debugging is twice as hard as writing a program in the first place. So if you’re as clever as you can be when you write it, how will you ever debug it?
> -- Brian Kernighan, 1974

Write easy to understand code, planning on future debugging.

# Communications Models
## Message Passing (modern)
* Puts messages into a shared queue, gives it a number, tell the other app "Go read this message"

## Shared Memory (ancient)
* Applications can just overwrite each others data.

# Scheduling
* **FCFS** - First come First Served. Not really used anymore
* **SJF** - Shortest Job first, kind-of how QoS works.
* **Priority** - Give processes an integer, rank them.
* **RR** - Round Robin, using time quantum, called q like 10-100 milliseconds
* **CFS** - *Completely Fair Scheduler
  * Involved, emulates time-slices
  * N tasks, each task gets 1/N time.


### Multilevel Queue - Done in Linux
* Foreground, Background
  * Foreground gets 80% as RR

* Background
  * FCFS
  
# Process Environment
* Argument vector - the command line arguments used to invoke the running program
* Environment vector - the list of "NAME=VALUE" pairs 

# Static and Dynamic Linking
* **Static** - the library functions are embedded in the executable.
* **Dynamic** - the library functions are at a place in memory, and shared.
