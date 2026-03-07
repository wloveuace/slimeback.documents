
In this chapter we will cover more thread related topics that didn't fit in the previous chapters.

## ==**Thread local storage**== 

In most modern operating systems and programming models, **threads within the same process** share some memory segments but have their own private ones. text , BSS, Data, Heap are shared (All threads of a process share its virtual address space including global and static variables ) while Stack is private for each thread. Despite Stack being private and having access to process global segments its always good to have another storage unit on Thread to Thread basis 

Thread Local Storage (TLS) is a user-mode mechanism that allows storing data on a per thread basis, accessible by each thread in the process, but only to its own data with uniform access method [^1]

**What does the TLS look like ?**  It is an array of LPVOID values which are initialized by default to zero, a thread can use it as an array where it uses an index to specify what slot to use (this index is global and can be seen by the whole process, but **stored values are per-thread**. ) , Usually these slots are used to save pointers to bigger data types like structs

![[Pasted image 20260306030006.png]]

TLS has 2 types of allocation , **Dynamic and static** , we will discuss both

Note: The constant `TLS_MINIMUM_AVAILABLE` defines the minimum number of static TLS indexes available in each process. This minimum is guaranteed to be at least 64 for all systems. The maximum number of indexes per process is 1,088.
### Dynamic TLS

Dynamic TLS allocation in Windows works by assigning a **process-wide TLS index** that each thread uses to store its own thread-specific pointer.

When using dynamic TLS allocation:

- The OS allocates a **TLS slot number**.
- The returned **index is global to the entire process**.
- **No memory block is allocated by the OS** at this stage; only the slot index is reserved.

Each thread already contains a **TLS pointer array** inside its Thread Environment Block(explained later)

The program then:
1. Allocates memory for the thread-specific data (typically on the heap).
2. Stores the pointer in the TLS slot.
3. Retrieves the pointer later using.

Internally, the index returned by `TlsAlloc` is used as an offset into the thread’s TLS array in the **TEB**, allowing each thread to store a different pointer in the same slot.

Note: When the OS returns an index, that index is guarantee to be free for all threads in the process meaning if a thread gets slot 15 as free then all threads in the process have that slot free aswell , if a thread has 15 occupied the OS returns another index.

```
TlsAlloc() - The function returns an available slot index and zeros all the corresponding cells for all existing threads to zero (removes junk form the cell for all threads)
```

```
TlsSetValue() - sets a value for the slot with index given
TlsGetValue() - gets value in slot with given index
TlsFree() - frees an index (must be used so other new created threads can use that index)
```
### Static TLS

TLS is “static” in the sense that it does not need any allocation, and it cannot be destroyed. Internally, the compiler bundles up all the thread-local variables into one chunk and stores the information in the PE in a section named `.tls`. The loader (NTDLL) that reads this information when the process starts up calls `TlsAlloc` to allocate a slot and allocates dynamically for each thread that starts up a memory block that contains all the thread-local variables.

The Microsoft-specific specifier `__declspec(thread)` can be used to designate a thread local variable like so:
```c
__declspec(thread) int counter;
```

The variable counter is now thread-local. Each thread has its own value.

Static TLS is useful for static TLS allocations without the need of `Tlsalloc`, everything is done automatically by the loader

## ==**Remote Threads**==

Windows provides under proper access for a process to create a thread in another process that can execute commands and communicate with caller process, this is a very useful concept in debugging and malware

```
CreateRemoteThread/Ex - Allow creating a thread in remote process (the handle must have create thread access also read and write access to VM )
```

The most interesting parameter is the function pointer itself (`lpStartAddress`). The address of the function is relative to the target process, meaning the code the thread needs to execute should already be there somehow. One idea is to use a function that is guaranteed to be in the target process and in a known address. Windows API functions are generally of this type. Since the Windows subsystem DLLs (kernel32.dll, kernelbase.dll, user32.dll, etc. and certainly ntdll.dll) are mapped to the same address in all processes, the address obtained from the calling process can be used in the target process as well. ( We generally use `GetModuleHandle` which returns the address of a loaded module in this process (kernel32.dll), and `GetProcAddress` retrieves a function’s address. )

Note:
The thread function prototype must be identical or near to:
```c
DWORD WINAPI ThreadFunction(PVOID param);
```

## ==**Thread Enumeration**==

We can enum threads in a process using same THELP api we used to enum for processes.

```
CreateToolhelp32Snapshot() - creates a snapshot handle
```
```
Thread32First() - get the first thread in the list
Thread32Next() - gets the next thread and moves the list pointer
```

```c
typedef struct tagTHREADENTRY32 { 
DWORD dwSize; // must be set before calls
DWORD cntUsage;
DWORD th32ThreadID; // this thread
DWORD th32OwnerProcessID; // Process this thread is associated with 
LONG tpBasePri;
LONG tpDeltaPri;
DWORD dwFlags;
} THREADENTRY32;
```

This is the returned structure that holds thread info

We wont talk much about this section since its relatively straight forward.

## ==**Caches and Cache Lines**==

In the early days of microprocessors, the CPUs’ speeds and the memory’s (RAM) speeds were comparable. Then CPU speeds went up and memory speeds lagged. This leads to a situation
where the CPU stalls a lot, waiting for memory to read or write a value. To compensate, a cache was introduced between the CPU and memory

The cache is a fast memory compared to main memory, which allows the CPU to stall less. Naturally, the cache is not nearly as large as main memory, but its existence is essential in today’s systems.

![[Pasted image 20260306074143.png]]

**How does a CPU use the cache ?** When a CPU reads data, it doesn’t read a single integer or whatever it was instructed to read ,but reads an entire cache line (typically 64 bytes) and places it in its internal cache. Then when reading the next integer in memory ,no memory access is required because the integer is already present in the cache.

Technically, there are 3 cache levels implemented in most CPUs. The closer the cache to the processor ,the faster it is and the smaller it is.

![[Pasted image 20260306074630.png]]

Level 1 cache is made up of a data cache (D-cache) and instruction cache (I-cache) and is per logical processor. Then there is cache level 2, which is shared by logical processors that are part of the same core. Finally, level 3 cache is system-wide.

There is yet another important cache not shown by Task Manager, called Translation Lookaside Buffer (TLB). This is a CPU cache dedicated to a fast translation of virtual to physical addresses. We’ll discuss this cache further in chapter 12.

Based on the info we discussed its better when accessing big data types like multi-dimensional array we should optimize our code to get the most out of the cache lines and respect their access for fast CPU reading, one of the most famous misuse of cache lines is **False sharing**

**False sharing** When a thread modifies a variable, the entire cache line containing that variable is marked as modified. If another thread accesses a different variable within the same cache line, the cache line is invalidated and must be reloaded from memory, even though the accessed variable was not modified. This results in increased memory traffic and reduced performance.

For example, consider two variables _A_ and _B_ stored in the same cache line. If thread 1 frequently updates _A_ while thread 2 reads _B_, the cache line will constantly be invalidated and reloaded, even though _B_ remains unchanged.

A fix for false sharing is **Align variables to separate cache lines** and using TLS

You maybe wondering **When does the CPU read from cache and when from main memory ?** 
The CPU tries to use the cache as much as possible leaving us with 2 scenarios
1. The CPU checks the fastest, smallest cache (L1) first. If the data is found, it is a **cache hit**. 
2. If the required data is not in L1, L2, or L3, a **cache miss** occurs and the CPU Reads from Memory (Cache Miss)

## ==**Wait Chain Traversal**==

If a deadlock does occur in a non-trivial application, it’s not easy to discover where the deadlock is. There are some techniques used with debuggers that can help locate such deadlocks, this is when WCT comes handy.

The WCT API provides the ability to traverse a wait chain starting from a thread of interest.

A wait chain contains an alternating sequence of threads and objects. Each thread in the A wait chain contains an alternating sequence of threads and objects. Each thread in the stream waits for an object that follows it, which is owned by the following thread in the chain, and so on.

Thread A (waiting for object) -> Thread B (owns object) **That's valid**
Thread A (waiting for object B, owns object A) -> Thread B (owns object B, waiting for object A) **That's a deadlock and we can detect it**

Wait chain analysis is able to track chains involving the following objects:
   • Critical sections
   • Mutexes (including across processes)
   • Asynchronous Local Procedure Calls (ALPC) - the internal inter-process communication mechanism used by Windows components
   • `SendMessage` - The `SendMessage` API is synchronous, and if called from a thread that is not the window’s owner causes the thread to block
   • Wait operations on threads and processes
   • Component Object Model (COM) cross-apartment calls (see chapter 18 for more on COM)
   • Sockets and Simple Message Block (SMB) operations

```
OpenThreadWaitChainSession() - The function opens a session for WCT and specifies whether the session should synchronous or asynchronous, returns an opaque handle to the WCT session.
```

Specifying zero for Flags sets a synchronous session. This means the thread that performs the analysis is blocked until the analysis is complete. Specifying `WCT_ASYNC_OPEN_FLAG` (1) indicates an asynchronous session, in which case the callback parameter should point to a function that is invoked when the analysis is complete.

```
GetThreadWaitChain() - a wait chain analysis is invoked for a specific thread with this function
```

`WctHandle` is the handle received from `OpenThreadWaitChainSession`. Context is an optional value that is passed as-is to the callback provided to `OpenThreadWaitChainSession` in case of an asynchronous session. Flags indicates which out-of-process should be considered, `ThreadId` is the thread from which to begin wait chain analysis.

Callback prototype:
```c
typedef VOID (CALLBACK *PWAITCHAINCALLBACK) (
 HWCT WctHandle,
 DWORD_PTR Context,
 DWORD CallbackStatus,
 LPDWORD NodeCount,
 PWAITCHAIN_NODE_INFO NodeInfoArray,
 LPBOOL IsCycle
 );
```

`NodeCount` points to the number of analysis nodes the function is willing to take. On return, it specifies how many actual nodes were written. The maximum analysis depth, which is the maximum nodes that can be returned is defined by `WCT_MAX_NODE_COUNT` (currently defined as 16). `NodeInfoArray` is the output array of node objects. Finally, the last output parameter, `IsCycle`, indicates whether there is a cycle in the analysis, returning TRUE if there is a deadlock

![[Pasted image 20260306081700.png]]

Node struct:
![[Pasted image 20260306081918.png]]

Each node represents one object in the chain. A chain always starts with a thread object and then is followed (if the thread is waiting on something) by an object, then by a thread (if that thread owns the object), etc.

`ObjectType` indicates what object is represented by the current node.
![[Pasted image 20260306082033.png]]

```
CloseThreadWaitChainSession() - close wait chain object
```

## ==**User Mode scheduling**==

The kernel scheduler is responsible for determining which thread should run on which processor, and make the context switches when needed. In some extreme cases, this is not as efficient as it can be. It would be advantageous in some scenarios to be able to control scheduling from user-mode rather than kernel mode. These decisions should not require a switch from user mode to kernel mode, as these switches are not cheap.
[^2]

Windows introduced User Mode Scheduling (UMS), where a user-mode thread becomes a scheduler of sorts and can schedule threads from user mode without the need to have a user-mode/kernel mode transition.

```
#include <ppl.h>
```

`ppl.h` provides the convenience functions, that handles user mode scheduling

```
parallel_for() - The parallel_for function does what it implies: a for loop, parallelized automatically. You just specify the initial value and the final value plus one, and a function to run for each iteration
```

Note there is no indication for how many threads to create or how to manage them.

## ==**Init Once Initialization**==

A **singleton object** means **there is only one instance of that object in the application**.

One common requirement for a singleton is to be initialized just once. In a multithreaded application, multiple threads may access the singleton at the same time initially, but the singleton must be initialized just once.

The One-Time Initializing API is the simplest version of synchronous initialization

A variable of type `INIT_ONCE` is used to control on-time initialization. It must be initialized in a static manner  (global or static variable) so that its initialization is guaranteed to be just once.

the simples form of initialization is
```
INIT_ONCE init = INIT_ONCE_STATIC_INIT;
```

an alternative is:
```
InitOnceInitialize() - initialize the singleton variable and returns a init once object pointer
```

```
InitOnceExecuteOnce() - does the initalization , it executes a callback function that does the work
```

```c
BOOL (WINAPI *PINIT_ONCE_FN) (
 _Inout_ PINIT_ONCE InitOnce,
 _Inout_opt_ PVOID Parameter,
 _Outptr_opt_result_maybenull_ PVOID *Context
 );
```

*We will skip the rest of the chapter as its not useful, better seek visual studio documents to know how to use its debugger*

## Functions discussed in this chapter:
```
TlsAlloc()
TlsSetValue()
TlsGetValue()
TlsFree()

CreateRemoteThread()

OpenThreadWaitChainSession()
GetThreadWaitChain()
CloseThreadWaitChainSession()

parallel_for()

InitOnceInitialize()
InitOnceExecuteOnce()
```


---
# Footer

[^1]: example for using TLS: the implementation of I/O functions such as `fopen `store the error result to the current thread using TLS.

[^2]: Windows provided fibers, which attempted to provide a user-mode scheduling mechanism .However, fibers were not recognized by the kernel, which caused a number of issues, such as Thread Local Storage not properly propagated, Thread Environment Block structures not aligned with the currently executing fiber and more. Fibers should not be used today.
