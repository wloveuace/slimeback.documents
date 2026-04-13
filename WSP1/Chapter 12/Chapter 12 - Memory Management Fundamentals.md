
we’ll introduce all major concepts related to memory- both virtual and physical

## ==**History of Virtual memory**==

In old systems and CPUs, Memory was relatively simple, as an application just allocated physical memory directly, used it, freed it, and that was it.

 Each access to memory was a combination of a segment address and an offset, which was needed because these processors worked with 16-bit values internally, but the memory access requires 20 bits (1 MB). A segment register’s value (16-bit) was multiplied by 16 (0x10) and then an offset was added to reach an address in the 1 MB range. This mode of working is now called Real Mode, and is still the mode in which today’s Intel/AMD processors wake up in.
 
 With the introduction of the 80386 processor, virtual memory was born as it’s essentially used today, including the ability to access memory linearly (with the segment registers just set to zero) by using offsets only

>On 64-bit systems, protected mode is called Long Mode,

## ==**What is Virtual memory**==

Virtual memory means every memory access needs to be translated to where the physical address is. This mode is referred to as Protected Mode. In protected mode, there is no way to access physical memory directly- only via a mapping from a virtual address to a physical address. This mapping must be prepared upfront by the operating system’s Memory Manager, since the CPU expects this mapping to be present.

> **Mapping** is the process of associating a virtual memory address with a physical memory address in RAM.

The mapping between virtual and physical addresses, as well as the management of memory blocks on the OS level are performed in chunks called pages. This is necessary, since it’s not possible to manage every single byte.

![[Pasted image 20260402202745.png]]

>Small (normal) pages are the default, and the term “page” used throughout this chapter (and the next ones) means a small or normal page, which is 4 KB on all architectures

## ==**Process Address Space**==

Each process has its own Private, linear, virtual, private address space. The address space starts at address zero and ends at some maximum value, based on the OS

How can multiple processes have same virtual address but different data ?
The answer is **private virtual memory**, For example, stating that there is some data at address 0x100000 requires another question answered: in which process? Every process has an address 0x100000, but that address may be mapped to a different physical address, to a file, or to nothing at all.

![[Pasted image 20260402220840.png]]

A process can directly access memory in its own address space. This means a process cannot accidentally or maliciously read or write to another process’ address space simply by manipulating a pointer. It is possible to access memory of another process, but that requires calling special functions.

>Every process starts with very modest usage of its virtual address space- the executable is mapped as well as `NtDll.Dll`. Then the loader (part of `NtDll`) allocates some basic structures within the process address space, such as the default process heap (discussed in the next chapter), the Process Environment Block (PEB), the Thread Environment Block (TEB) for the first thread in the process. Most of the address space is empty.


## ==**Page states**==

Each page in virtual memory can be in one of three states:
- free 
- committed
- reserved.

**Free pages** are unmapped, and so trying to access a free page causes an access violation exception.

**Committed pages** are the opposite of free pages. A committed page has reserved **backing storage** (either in RAM or on disk, such as a page file or mapped file). This means accessing the page is guaranteed to succeed, but it does **not** mean the page is currently in RAM. Instead, the OS guarantees that the page can be placed in RAM when needed because space has been reserved (in RAM or disk).

If the page is already in RAM (i.e., **resident**), the CPU accesses the data directly and continues execution. If the page is not in RAM, the CPU cannot access it directly (it cannot read from disk), so it raises an exception called a _page fault_. The memory manager handles this by bringing the page into RAM (either by allocating a new frame or loading it from disk), updating the mapping, and then the CPU retries the instruction successfully.

>Committed memory is what normally is called “allocated” memory. Calling C/C++ memory allocation functions, such malloc, calloc, operator new, etc

**Reserved pages** are similar to free, in the sense that accessing that page causes an access violation, but it also ensures that normal memory allocations do not use the range that its been specified because it’s reserved for another purpose.

>We’ve seen this idea in the way a thread’s stack is managed. Since a thread’s stack can grow, and must be contiguous in virtual memory, a range of pages is reserved so that other allocations happening in the process don’t use the reserved address range

## ==**32-bit Systems**==

32-bit means 4 GB, which may have left you wondering why processes get only 2 GBs of virtual address space or 3 GBs MAX ?  The virtual memory is split into 2 , **2 GB user space**, **2 GB kernel space** or 3 : 1. 

![[Pasted image 20260402223455.png]]

>System-reserved virtual memory stores the OS kernel, drivers, memory management structures, and hardware mappings, and is shared across all processes but only accessible in kernel mode.

>Even though System drivers & kernel are singletons but like DLLs, the kernel and drivers exist once in physical memory, and each process’s virtual address space maps to the same physical pages, not copies.

## ==**64-bit Systems**==

The theoretical limit for 64 bits is 2 to the 64ʰ power, or 16 EB,  This is a literally astronomical address range, that seems unreachable in today’s systems. (Limit for 32bit was 4gb).

>Most modern processors support only 48 bits of virtual and physical addresses.

![[Pasted image 20260402225035.png]]

## ==**Address Space Usage**==

To get a sense of what Memory is usable and what's not , We can use the following API

```c
VOID GetNativeSystemInfo(
  [out] LPSYSTEM_INFO lpSystemInfo
);
```
To retrieve accurate information for an application running on WOW64
Parameters:
	- `lpSystemInfo` - A pointer to a [SYSTEM_INFO](https://learn.microsoft.com/en-us/windows/desktop/api/sysinfoapi/ns-sysinfoapi-system_info) structure that receives the information.

![[Pasted image 20260402225653.png]]

>WOW64 is a subsystem that allows 32-bit applications to run on 64-bit Windows.

You can check if a process is WOW64 using:
```c
BOOL IsWow64Process(
  [in]  HANDLE hProcess,
  [out] PBOOL  Wow64Process
);
```
Determines whether the specified process is running under [WOW64](https://learn.microsoft.com/en-us/windows/desktop/WinProg64/running-32-bit-applications) or an Intel64 of x64 processor.
Parameters:
	- `hProcess` - Handle to the process
	- `Wow64Process` - A pointer to a value that is set to TRUE if the process is running under WOW64

>The newer `IsWow64Process2` function provides more information about the processor powering the process and the native processor on the machine. `pProcessMachine` returns one of the `IMAGE_FILE_MACHINE_*` constants defined in `<winnt.h>`


## ==**Memory counters**==

The system keeps track of alot of memory related info that can be usefull for programmers to track the memory usage on the OS/a specific process 

![[Pasted image 20260412155628.png]]

The task manger and operating system categorize memory pages by the following:
- **In use**: These are pages currently considered part of processes and the system’s working set
- **Standby pages**: Those are pages that are **currently in RAM but not part of any process’s working set** Their contents are **still valid** and backed by disk (file or pagefile)(safe to discard the RAM copy if needed), so they can either be:
	  **Quickly restored to a process** if accessed again (soft page fault), or
	  **Reused for other purposes** if memory is needed.
- **Modified**: Those are pages that haven't been backed up in a page file yet
- **Zeroed**: Those are pages containing zeros only
- **Free pages**: Those are pages that have junk

>The reason zero pages are important, is to satisfy a security requirement, where allocated memory cannot ever include data once belonging to another process, even if that process no longer exists.

> **Backing = where the page can be reconstructed from if it is not in RAM**
>  “If we remove this page from memory, where do we get it again?”
>  
> Types of backing:
> 	- File-backed pages: The **original file on disk** such as .exe code , DLLs
> 	- Pagefile-backed pages (private memory): OS writes page to pagefile (`pagefile.sys`) then later gets the page back , example: `heap`, `stack`

>A page being part of a **working set** means:
		**It is currently mapped into a process’s virtual address space and actively usable by the CPU without causing a page fault.**

Memory priority is used when pages from the standby list need to be moved to become free pages, because processes or the system need physical memory. If physical memory is needed, their standby pages should be the first to go, even if they were relatively recently used. This is where memory priority comes in.

When a process requires memory and there isnt enough memory pages , the pages with lower priority are recycled and reallocated and given to the process
- **Low-priority standby pages** → reclaimed _first_
- **High-priority standby pages** → kept longer (more likely to be reused soon)
  
```
GetProcessInformation
SetProcessInformation

GetThreadInformation
SetProcessInformation
```

those are the functions used to set/get memory page priority where the value of the priority is between 1 and 5. This means only lowering memory priority is permitted. (5 = Normal priority , 1 = Very Low)

>The above “set” functions are thin wrappers over the native `NtSetInformationProcess` and `NtSetInformationThread` functions. Similarly,  “get” functions are thin wrappers around `NtQueryInformationProcess` and `NtQueryInformationThread`.

## ==**Process Memory Map**==

A process’ address space must contain everything that is used by the process in terms of memory: The executable’s code and global data, DLLs code and global data, Threads stacks, heaps, and any other memory committed and/or reserved by the process

![[Pasted image 20260413161515.png]]

This is a minimal process map. It can also be looked at by the following sections in the diagram below:
![[Pasted image 20260413162145.png]]

> Mapped Files are like DLLS that are backed by a file page
> Shareable are like shared memory pages that are backed by a PageFile  

To get the name of a mapped file we can use
```c
DWORD GetMappedFileNameA(
  [in]  HANDLE hProcess,
  [in]  LPVOID lpv,
  [out] LPSTR  lpFilename,
  [in]  DWORD  nSize
);
```
Checks whether the specified address is within a memory-mapped file in the address space of the specified process. If so, the function returns the name of the memory-mapped file.
Parameters:
	`hProcess` - process handle
	`lpv` - address of the memory mapped file
	`lpFilename` - buffer to hold name
	`nSize` -  number of characters written

we can get the process memory information using:
```c
BOOL GetProcessMemoryInfo(
  [in]  HANDLE                   Process,
  [out] PPROCESS_MEMORY_COUNTERS ppsmemCounters,
  [in]  DWORD                    cb
);
```
![[Pasted image 20260413163142.png]]

>The kernel has two basic memory pool types: paged pool, which holds memory that can be paged to disk, and non-paged pool, which by its definition is always resident in RAM, and is never paged out to disk. These memory pools are used by the kernel and device drivers

## ==**Page Protection**==

Every committed page in a process virtual address space has protection flags. These can be set with the `VirtualAlloc` or `VirtualProtect` functions

Any access that violates a page’s protection causes an access violation exception.
![[Pasted image 20260413163943.png]]

![[Pasted image 20260413164231.png]]

Page protection is set initially when calling `VirtualAlloc` for new allocations, and can be changed by calling `VirtualProtect` for exiting pages.

## ==**Enumerating Address Space Regions**==

when enumerating pages they are clumped in *regions*

A **region** = one or more **adjacent pages** that all share the same:
- State (e.g., MEM_COMMIT, MEM_RESERVE, MEM_FREE)
- Protection (e.g., PAGE_READWRITE)
- Type (e.g., MEM_PRIVATE, MEM_MAPPED, MEM_IMAGE)

So the OS groups pages together into regions for efficiency.

We can retrieve information about a range of pages in the virtual address space of a process.
```c
SIZE_T VirtualQueryEx(
  [in]           HANDLE                    hProcess,
  [in, optional] LPCVOID                   lpAddress,
  [out]          PMEMORY_BASIC_INFORMATION lpBuffer,
  [in]           SIZE_T                    dwLength
);
```
Parameters:
	`hProcess` - Handle to process
	`lpAddress` - optional address to the base address of the region of pages to be queried (The address is always rounded down to the nearest page boundary.)
	`PMEMORY_BASIC_INFORMATION` - pointer to buffer that will hold  `MEMORY_BASIC_INFORMATION`
	`dwLength` - set to size of `MEMORY_BASIC_INFORMATION`

![[Pasted image 20260413164905.png]]

>To query all pages we can set address to 0 and then increment by `RegionSize` to get to the next region 

## ==**Sharing Memory**==

it’s sometimes beneficial to be able to share memory between processes. The canonical example is DLLs. All user-mode processes need NtDll.dll, and most need Kernel32.Dll, KernelBase.dll, AdvApi32.Dll, and many others. If each process had its own copy of the DLL in physical memory, it would quickly run out. In fact, one of the primary motivations to have DLLs in the first place, is the ability to share (at least their code). Code is by convention read-only, so can safely be shared. The same goes for executable code coming from an EXE file. If multiple processes execute based on the same image file, there is no reason not to share (at least the code).

![[Pasted image 20260413182813.png]]**Shared memory & mapping**

Kernel32.DLL code is places in a place in physical ram and mapped in the virtual process of any process that uses it , same for the image code

since code of image is fixed , What about global data and what will happen if we run 2 instances?
	 If we declare a variable in global scope like so: `int x; void main() { x++;}` , If we run the instance twice The answer is x is global to a process, not to the system. This works the same with a DLL. If a DLL declares a global variable, it’s only global to each process the DLL is loaded into.

In most cases, this is what we want. This works by utilizing a page protection called Copy on Write (PAGE_WRITECOPY). The idea is that all processes using the same variable (declared in the executable or in a DLL used by these processes) map the page this variable is located to the same physical page, If a process changes the value of that variable an exception is thrown, causing the memory manager to create a copy of the page and hand it to the calling process as a private page, removing the Copy-on-Write protection

![[Pasted image 20260413183554.png]]

So for Regular global variables in a DLL/EXE:
- Initially shared
- When written → **private copy is created per process**
✔ This saves memory  
✔ This is what the paragraph means by:
	“If the data isn’t changed, no copy needs to be made”
  
  
But what if we don't want to copy the memory for every edit made by the process , we can make a shared memory section that holds variables that are global for every process , **editing the variable by a process will change the value for everyone , no private copy is made**

We can use the linker to create a new *global* section in the PE.
```
#pragma data_seg("shared")
int x = 0;
#pragma data_seg()

#pragma comment(linker, "/section:shared,RWS")
```
- **Shared** is the section name , can be any 8 letters name or less
- Variables are placed between the initialization of the section and the closing of it
- Variables must be initialized explicitly
- We can add a comment to change the section access rights `"/section:SectionName, Rights"`
	RWS (read, write, shared)

## ==**Page Files**==

Processors can only access code and data in physical memory (RAM). If some executable is launched, Windows maps the executable’s code and data (and NTdll.dll) into the process’ address space. Then, the process’ first thread starts execution. This causes the code it executes (first in NtDll.dll and then the executable) to be mapped to physical memory and loaded from disk so that the CPU can execute it.

Suppose that process’s threads are all in a wait state, perhaps the process has a user interface, and the user minimized the application’s window and didn’t work with the application for a while. Windows can repurpose the RAM used by the executable for other processes that need it. Now suppose the user restores the application’s window- Windows now must bring back the application’s code into RAM. Where would the code be read from? The executable file itself.

This means executables and DLLs are their own backup. In fact, Windows creates a Memory
Mapped File for executables and DLLs (which also explains why such files cannot be deleted
since there is at least one open handle to the files)

What about data? If some data is not accessed for a long time (or Windows is low on free memory), the memory manager can write the data to disk- to a page file. A page file is used as backup for private, committed memory. Using a page file is not required- Windows can function just fine without one. But this reduces the amount of memory that can be committed at a time.

Windows supports up to 16 page files. They must be in different disk partitions and are named pagefile.sys, located at the root’s partition (the files are hidden by default). Having more than one page file may be beneficial if one partition is too full, or another partition is a separate physical disk, which can increase I/O throughput.

Each page file can have an initial size and a maximum size. If the system reaches its commit limit, the page files are increased to their configured maximum value, so the commit limit is now increased (at the possible performance degradation because of more I/O). If the committed memory drops below the original commit limit, the page file sizes will reduce back to their initial sizes.

>suppose some user requires 40 GB of committed memory for her work. If the machine has 8 GB of RAM, then the page file size should be set to around 32 GB.
>Of course having more RAM is beneficial, since it reduces the likelihood of page file usage, but the page file size has nothing to do with the amount of RAM on the system.

## ==**Virtual Address Translation**==

we’ll look at the basics of how virtual addresses are translated into physical addresses. This section is strictly optional, and can be skipped entirely.


`mov eax, [100000H]`
When we do such an instruction the CPU already knows it a virtual address rather than physical (since the CPU is configured to run in protected mode / long mode). 

The CPU must now look at tables that were prepared beforehand by the memory manager, that describe where that page is in RAM, if at all. If it’s not in RAM it raises a page fault exception, to be handled appropriately by the memory manager

![[Pasted image 20260413213907.png]]

The CPU is provided with a virtual address as input, and should output (and use) a physical address. Since everything works in terms of pages, the lower 12 bit of an address (the offset within a page) are never translated, and pass as-is to the final address.

To translate between the two, the system uses a hierarchy of tables (often called a _translation tree_). Instead of one huge table, the address is resolved step by step through multiple levels. On modern systems, the CPU starts from a top-level structure (like the PML4 *page map level 4* on 64-bit systems), then follows pointers through lower levels such as the page directory and finally the page table.

The **page directory** sits in the middle of this process. Its job is not to give the final physical address, but to **guide the CPU to the next step**. Each entry in the page directory (called a Page Directory Entry, or PDE) typically points to a **page table**, which is where the actual mapping to physical memory happens. In some cases, the page directory can skip that step and directly map a large block of memory (a “large page”), but most of the time it simply says: “to resolve this address, go look in this specific page table.”

the **page table entries (PTEs)** are the ones that directly point to physical memory. When a page is moved to the page file, the operating system marks the corresponding PTE as **invalid**. The page directory is still valid and still points to the correct page table, but when the CPU reaches the final step and finds that the PTE is invalid, it cannot complete the translation. At that point, it raises a **page fault exception**, which tells the OS: “I tried to access this memory, but it’s not currently in RAM.” The OS then loads the page back from disk, fixes the PTE, and the program continues.

```
Virtual Address
   ↓
Top-level table
   ↓
Top-level entry
   ↓
Page Directory
   ↓
Page Directory Entry (PDE)
   ↓
Page Table
   ↓
Page Table Entry (PTE)
   ↓
Physical Memory
```

Finally, the Translation Lookaside Buffer (TLB) is a cache of recently translated pages, so accessing these pages does not require going through multiple levels of structures for translation purposes. This cache is relatively small and is very important from a practical perspective. This emphasizes working with the same range of memory address at close times is great for utilizing the TLB cache.