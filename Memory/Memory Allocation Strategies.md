
This is based on the series of articles made by Sir Ginger Bill 
[Memory Allocation Strategies - gingerBill](https://www.gingerbill.org/series/memory-allocation-strategies/)

This summary will include the whole article simplified and in one paper

## Article 1 - ==Thinking About Memory and Allocation==

Memory allocation seems to be struggling to understand , specially when you have the mentality of "Memory is the stack and the heap" , This is completely wrong thinking of the memory.

Modern operating systems think of memory as a Per process basis . Each process has its own liner address space that doesn't interfere with any of the  memory of another process ( some cases do ). This concept is called *Virtual Memory*. The virtual memory is then categorized into different sections Some of that virtual address space is reserved for procedure stack frames, some of it is reserved for things required by the operating system, and the rest we can use for whatever we want.

When it comes to allocation, there are three main aspects to think about:
- size of allocation
- lifetime of the allocation
- usage of the memory
  
Those 3 aspects are the main contributors causing different memory allocation algorithms to exist as each one is made based on a specific purpose 

![[Pasted image 20260422231620.png]]

This table explains most of the cases when it comes to memory allocation. 
Case 1 : Most obvious case that usually occurs ( lifetime of the allocation and size are known factors)
Case 2 : Common for some examples such as loading a file into memory at runtime and populating a hash table of unknown size ( Lifetime known , Size unknown )
Case 3 : This is the case when Size of a memory allocation is known but lifetime is unknown , usually happens when having shared memory (Lifetime unknown, size known)
Case 4: This is the case where the garbage collectors act on (Size unknown , lifetime unknown)

Different life times have different usage cases , that's why there are categorized memory allocations based on the lifetime also called *Allocation generation* (An allocation generation is a way to organize memory lifetimes into a hierarchical structure) :
- **Permanent Allocation** : The allocation isn't freed and is valid until the end of the program 
- **Transient Allocation**: Memory that has a cycle-based lifetime. This memory only persists for the “cycle” and is freed at the end of this cycle.
- **Scratch/Temporary Allocation**: Short lived, quick memory that I just want to allocate and forget about , common case a short buffer or stack allocated variables within a block

In languages with automatic memory management, many people assume that the compiler knows a lot about the usage and lifetimes of your program. **This is false**. You know much more about your program than the compiler could ever know.
The language struggles to know when it should pre-allocate or free in bulk. This is compiler ignorance can lead to a lot of performance issues.

## Article 2 - ==Liner / Arena allocator==

*To be continued*

