
### ==Windows Architecture Overview 

 ==**Process**== is a containment and management object that represents a running instance of a program, process contain threads that run the code and process act as a container holding objects related to the process and threads in it. Process usually contain `atleast 1 thread, Private virtual address space, access token ( security related ) , private handle table`

`Process abstraction - diagram 1`
![[operating systems/WSP1/Chapter 1/d2.png]]

==**Process identification**== is identified by a Process id ,which remains unique as long as the kernel process object exists. Once it’s destroyed, the same ID may be reused for new processes.

==**Dynamic Link Libraries**== (DLLs) are executable files that can contain code, data and resources. DLLs are loaded dynamically into a process either at process initialization time (called static linking) or later when explicitly requested (dynamic linking). DLLs allow sharing their code in physical memory between multiple processes that use the same DLL( this process is called Mapping , later explained in chp12 )[^1]

`DLL sharing - diagram 2`
![[operating systems/WSP1/Chapter 1/d1.png]]

==**Virtual Memory**== Every process has its own virtual, private, linear address space. This address space starts out empty `NTDLL is normally the first thing to be mapped in the process virtual memory as it runs some functions and does some intialization` Once execution of the main (first) thread begins, memory is likely to be allocated, more DLLs loaded, etc. This address space is private, which means other processes cannot access it directly. The address space range starts at zero (although technically the first 64KB of address cannot be allocated), and goes all the way to a maximum ` for x86 process address space is 2GB by default and can go up to 3bg, on x64 the address space is 8TB and can go upto 128TB`

`The memory itself is called virtual, which means there is an indirect relationship between an address range and the exact location where it’s found in physical memory (RAM). A buffer within a process may be mapped to physical memory, or it may temporarily reside in a file (such as a page file). The term virtual refers to the fact that from an execution perspective, there is no need to know if the memory about to be accessed is in RAM or not; if the memory is indeed mapped to RAM, the CPU will access the data directly. If not, the CPU will raise a page fault exception that will cause the memory manager’s page fault handler to fetch the data from the appropriate file, copy it to RAM, make the required changes in the page table entries that map the buffer, and instruct the CPU to try again.`
[^2]
`
==**Threads**== are the actual entities that execute code are threads. A Thread is contained within a process, using the resources exposed by the process to do work (such as virtual memory and handles to kernel objects). Threads have some private attributes: `Current access mode, CPU Execution context, Stack for local variables, Thread Local Storage (TLS) array for private thread data, Base and dynamic priority, Processor affinity`, those attributes are changed for each thread. Thread states ( RUNNING, READY , WAITING), 

```
Running - currently Executing
Ready - waiting for its turn in execution
Waiting - waiting for a specific event or object to be signaled
```

`Windows system general diagram - diagram 4`
![[operating systems/WSP1/Chapter 1/d3.png]]

```
User processes - These are normal processes based on image files

Subsystem DLLs - Subsystem DLLs are dynamic link libraries (DLLs) that implement the API of a subsystem

NTDLL.DLL - Asystem-wide DLL, implementing the Windows native API. This is the lowest layer of code which is still in user mode.

Service Processes - Service processes are normal Windows processes that communicate with the Service Control Manager (SCM, implemented in services.exe) and allow some control over their lifetime.

Executive - The Executive is the upper layer of NtOskrnl.exe (the “kernel”). It hosts most of the codethat is in kernel mode.Itincludes mostlythevarious“managers”

Kernel - The Kernel layer implements the most fundamental and time-sensitive parts of kernel mode OS code. This includes thread scheduling, interrupt and exception dispatching and implementation of various kernel primitives such as mutex and semaphore.

Device Drivers - Device drivers are loadable kernel modules. Their code executes in kernel mode and so has the full power of the kernel.

Win32k.sys - The kernel-mode component of the Windows subsystem. Essentially this is a kernel module (driver) that handles the user interface part of Windows and the classic Graphics Device Interface (GDI) APIs.

Hardware Abstraction Layer (HAL) - The HALisanabstraction layer over the hardware closest to the CPU. It allows device drivers to use APIs that do not require detailed and specific knowledge of things like an Interrupt Controller or a DMA controller

System Processes - System processes is an umbrella term used to describe processes that are typically “just there”, doing their thing where normally these processes are not communicated with directly, Example system processes include Smss.exe, Lsass.exe, Winlogon.exe, Services.exe and others.

Subsystem Process - The Windows subsystem process, running the image Csrss.exe, can be viewed as a helper to the kernel for managing processes running under the Windows system. It is a critical process, meaning if killed, the system would crash.

Hyper-V Hypervisor - The Hyper-V hypervisor exists on Windows 10 and server 2016 (and later) systems if they support Virtualization Based Security (VBS). VBS provides an extra layer of security, where the actual machine is in fact a virtual machine controlled by Hyper-V.
```

[^1]: Dlls arent copied to each process , dlls stay in some physical memory address and then mapped to the process virtual address 

[^2]: Virtual memory is intended to make the process think it has its owns all the RAM (  the process doesnt need to know if a buffer is saved in disk or mapped in physical memory, in process prespective any data is saved in RAM and is ready for use at any time) , provide a liner address space, make it private to the process threads to use, Virtual pages can be saved in disk and when needed for access its loaded in RAM ( saving RAM space ), Virtual address spaces are translated to physical address spaces via the MMU ( Memory management unit )
