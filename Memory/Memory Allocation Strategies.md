
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

The first memory allocation strategy that I will cover is also one of the simplest ones, Liner allocator also known as *Arena allocator* has a very basic logic and structure.

Consists of a contiguous block of memory , a base pointer , an offset the moves to the next free position
![[Pasted image 20260423093000.png]]

allocating memory is of complexity O(1) since it involves just moving the offset forward
![[Pasted image 20260423093018.png]]

Due to being the simplest allocator possible, the arena allocator does not allow the user to free certain blocks of memory. The memory is usually freed all at once.
Ex: You cant free from address (base+10) till (base+15) 

This allocator fits best for temporary allocations that get filled once and deleted after

```c
#include <stdio.h>
#include <stdlib.h>

#define POOL_SIZE 1024

typedef struct allocator {
	void* basePtr;
	size_t offset;
}Allocator;

void arenaAlloc(Allocator* allocObj, size_t allocSize) {
	allocObj->offset = (allocSize + allocObj->offset) % POOL_SIZE; //ring-buffer
	printf("Allocated: 0x%lu\n", allocSize);
	printf("Next free address: 0x%lu\n", (uintptr_t)allocObj->basePtr + allocObj->offset);
}

void arenaDealloc(Allocator* allocObj, size_t allocSize) {
	allocObj->offset = (allocObj->offset + 1) - allocSize ? allocObj->offset - allocSize : 0;
	printf("Next free address: 0x%lu\n", (uintptr_t)allocObj->basePtr + allocObj->offset);
}

int main() {
	Allocator* MemoryPool = (Allocator*)malloc(sizeof(Allocator) + POOL_SIZE);

	MemoryPool->offset = 0;
	MemoryPool->basePtr = (void*)(MemoryPool + 1);

	printf("Base ptr: 0x%lu\n", (uintptr_t)MemoryPool->basePtr);

	arenaAlloc(MemoryPool, 100);
	arenaDealloc(MemoryPool, 10);

}
```

This approach has an issue which is *memory alignment* specially when reading data

#### **Memory alignment**

Modern computer architectures will always read memory at its “word size” (4 bytes on a 32 bit machine, 8 bytes on a 64 bit machine). If you have an unaligned memory access (on a processor that allows for that), the processor will have to read multiple “words”. This means that an unaligned memory access _may_ be much slower than an aligned memory access.

If you would like to learn more, I recommend the following articles:

- [IBM - Data alignment: Straighten up and fly right](https://developer.ibm.com/technologies/systems/articles/pa-dalign/)
- [Gallery of Processor Cache Effects](http://igoro.com/archive/gallery-of-processor-cache-effects/)
- [x86 Protected Mode Basics](http://www.rcollins.org/articles/pmbasics/tspec_a1_doc.html)

On virtually all architectures, the amount of bytes that something must be aligned by must be a power of two (1, 2, 4, 8, 16, etc). To align a memory address to the specified alignment is simple modulo arithmetic. You are looking to find how many bytes forward you need to go in order for the memory address is a multiple of the specified alignment.

The formula of checking alignment would be checking with mod operator , if the remainder is 0 then its aligned , if not then the amount of extra bytes we need to read would be alignment-remainder

```c
void arenaAlign(size_t* allocSize) {
	int rem = *allocSize % 8;
	if (rem != 0) {
		*allocSize += 8 - rem;
		printf("padded: %u\n", 8 - rem);
	}
}
```

*This is shitty code, optimized version is on my github*
*The offset should reset to zero when freed, but here we remove with a certain size*

## Article 3 - ==Stack Allocator==

In this article we will cover the stack allocator. This allocator follows the *LIFO* (LAST IN FIRST OUT) principle of the Stack data structure (this has nothing to do with the stack frame and function stack)

The stack allocator is a evolved version of the arena allocator with using the LIFO principle. We still have the same Offset and base pointer and the offset moves forward linearly. What's new is that we will contain more information about the allocation + the stack principle so we can deallocate the last allocated memory instead of freeing the whole pool or arbitrary size.

![[Pasted image 20260428001534.png]]

In order to keep information about the allocation made like its size , we can store a small header before the allocation and it will hold critical information thats used for freeing efficiently

To free a block, the header that is stored before the block of memory can be read in order to move the offset backwards. In Big-O notation, the freeing of this memory has complexity of _**O(1)**_ (constant).

![[Pasted image 20260428001756.png]]

#### The header

As we said, the header will hold all the critical data for each allocation.
- Store the padding from the previous offset
- Store the previous offset
- Store the size of the allocation

**What is the use for padding ?** in the previous Arena allocator , we had to add padding so that the allocation size is a multiple of 8 so that when we are reading/writing we dont have any issue(think of it as having multiple 8 bytes chunks). Here we dont need that no more; However, we need to add padding before each header so we make sure that we can read it fine and clean. The padding member stores the amount of bytes that has to be placed before the header in order to have the new allocation correctly aligned.

![[Pasted image 20260428002555.png]]

*The stack allocator is the first of many allocators that will use the concept of a header for allocations.*
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>

#define POOL_SIZE 1024

typedef struct {
    void* basePtr;
    void* dataPtr;
    size_t allocSize;
    int padding;
} Header;

size_t offset = 0;
int allocChunks = 10;
int freeMemory = POOL_SIZE + sizeof(Header) * 10;

Header* initStack() {
    Header* header = (Header*)malloc(freeMemory);

    printf("=== initStack ===\n");
    printf("Allocated total memory: %d bytes\n", freeMemory);
    printf("Base address: %p\n", header);

    header->basePtr = (void*)header;

    offset = sizeof(Header);
    header->allocSize = 0;
    header->dataPtr = NULL;
    header->padding = 0;

    printf("Initial offset: %zu\n", offset);
    printf("=================\n\n");

    return header;
}

Header* stackAlloc(Header* header, size_t allocSize) {
    printf("=== stackAlloc(%zu) ===\n", allocSize);

    if (!allocChunks || allocSize + sizeof(Header) > freeMemory) {
        printf("Invalid size or no chunks left\n\n");
        return NULL;
    }

    unsigned char* base = (unsigned char*)header->basePtr;

    printf("Current offset: %zu\n", offset);
    printf("Free memory before: %d\n", freeMemory);

    Header* newHeader = (Header*)(base + offset);

    printf("New header at: %p\n", newHeader);

    newHeader->allocSize = allocSize;
    newHeader->padding = 0;

    newHeader->dataPtr = (void*)((unsigned char*)newHeader + sizeof(Header));

    printf("Data pointer: %p\n", newHeader->dataPtr);
    printf("Header size: %zu\n", sizeof(Header));
    printf("Total block size: %zu\n", sizeof(Header) + allocSize);

    offset += sizeof(Header) + allocSize;
    freeMemory -= sizeof(Header) + allocSize;
    allocChunks--;

    printf("New offset: %zu\n", offset);
    printf("Free memory after: %d\n", freeMemory);
    printf("Remaining chunks: %d\n", allocChunks);
    printf("========================\n\n");

    return newHeader;
}

int main() {
    Header* start = initStack();

    Header* a = stackAlloc(start, 100);
    Header* b = stackAlloc(start, 50);
    Header* c = stackAlloc(start, 200);

    return 0;
}
```

We still need to add padding for alignment