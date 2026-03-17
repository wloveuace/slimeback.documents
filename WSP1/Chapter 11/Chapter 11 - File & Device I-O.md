
In previous chapters, we used threads in various ways to perform CPU-bound work. However, not all operations are CPU related. Some require communication with files or other devices, commonly known as I/O operations. These operations do not require CPU usage until the operations complete, at which point thread code continues processing with the result of the I/O operation.

## ==**The IO system**==

The main purpose of the I/O system is to abstract access to physical and logical devices. Accessing a file in the any file system should be different than accessing a serial port, a USB camera or a printer. The I/O System is comprised of multiple components, some in user-mode and most in kernel mode. 

`APP -> IO MANGER -> DRIVER -> HAL`

![[Pasted image 20260316103728.png]]

All file and device operations on the kernel side are initiated by the I/O manager. A request, such as read or write, is handled by creating a kernel structure called I/O Request Packet (IRP), filling in the details of the request and then passing it to the appropriate device driver. 

For real files, this goes to a file system driver, such as NTFS. This process is not essentially different than normal system calls

`APP -> IO MANGER -> IRP -> Driver -> HAL`

![[Pasted image 20260316103808.png]]

As far as the kernel is concerned, I/O operations are always asynchronous. This means that a driver should initiate the operation and return as soon as possible so that the calling thread can regain control. The original caller, however, can choose to make the call synchronous. In that case, the I/O manager waits on behalf of the caller until the operation is done. This flexibility is very convenient from the client’s perspective.

## ==**Files & Devices**==

```
CreateFile() - Create/Open a file object
```
The term “File” used in `CreateFile` is short for “File Object”, which is the abstraction used by the kernel to represent a connection to a device, whether that device happens to be a file in a file system or not

The `lpFileName` parameter indicates which file or device name to create or open. This is not necessarily a “file name”, as indicated by the parameter’s name. It is a symbolic link into the executive’s object manager namespace, with some parsing rules added.

![[Pasted image 20260316104148.png]]

The `dwDesiredAccess` parameter is used to specify the access mask required for accessing the file object. In most cases, you’ll use one or more of the generic access rights

You can also use more fine grained access masks relevant to the file or device being accessed. For example, `FILE_READ_ DATA` is a specific access mask for files, requesting reading their contents.

The `dwCreationDisposition` parameter specifies how to create or open file system objects (files and directories). For other devices, the flag should always be set to `OPEN_EXISTING`. (allows setting three separate flags/values that can be combined by the normal OR operator)

When a program opens a file using `CreateFile`, it can pass **flags** that tell Windows **how the file will be accessed** so the OS can optimize performance.
	`FILE_FLAG_NO_BUFFERING` This **disables the Windows file cache completely**
		Because Windows isn't helping with caching, **your program must respect the disk sector structure**.
		1. Because Windows isn't helping with caching, **your program must respect the disk sector structure**. (likely 512, 4096 bytes ) `GetDiskFreeSpace()`
		2. Memory buffers must start at **sector-aligned addresses**. (`_aligned_malloc`)

```
FlushFileBuffers() - Forces data in cache to be written to file
```

Typically the `WriteFile` and `WriteFileEx` functions write data to an internal buffer that the operating system writes to a disk or communication pipe on a regular basis. The  `FlushFileBuffers` function writes all the buffered information for a specified file to the device or pipe.

Buffering `API -> Internal Buffer -> Disk`
Non-buffering `API -> Disk`
### **Working with Symbolic Links**

A symbolic link is **just a name or path that points to something else** — it can be a file, folder, or device. The OS uses it to let you work with a friendly name while the real target may be somewhere else. Basically just a shortcut name to a longer path

As we’ve seen, `CreateFile` internally works by parsing symbolic links, we can get those links using `QueryDosDevice`

```
QueryDosDevice() - Retrieves information about MS-DOS device names. The function can obtain the current mapping for a particular MS-DOS device name. The function can also obtain a list of all existing MS-DOS device names.
```

if `lpDeviceName` is not NULL, the function looks up the symbolic link and returns its target (if any) in `lpTargetPath`. If `lpDeviceName` is NULL, it returns all symbolic links in `lpTargetPath`, separated by '\0', so they can be iterated if desired

If the function fails because the target buffer is too small, `GetLastError` returns `ERROR_INSUFFICIENT_BUFFER`.

![[Pasted image 20260316111650.png]]
*Symbolic links from winobj*

```
DefineDosDevice() - create a new symbolic link
```
`lpDeviceName` is the symbolic link’s name and `lpTargetPath` is the target of the link.

### **Path Length**

The file name length to CreateFile is traditionally limited to MAX_PATH characters, defined as 260. Windows 10 version 1607 and Windows Server 2016 added a new feature that allows breaking out of these path length limitations. To apply this feature we need 2 settings
- A global registry value named `LongPathsEnabled`  at `HKLM\System\CurrentControlSet\`
- **Control\FileSystem must be set to 1** (DWORD value). The first time an I/O function is called for a process, this value is read and cached for the lifetime of the process. This means any change to this value will be noticed for new processes only. If this value is changed and the system is rebooted, all processes are guaranteed to see the new value.

### **Directories**

The `CreateFile` function can open a handle to an existing directory if the flag `FILE_ FLAG_BACKUP_SEMANTICS` is specified in the `dwFlagsAndAttribute` argument.

```
CreateDirectory/Ex() - Create a new directory (it can have a full path or a relative path)
```
Ex version allows specifying an existing directory from which some properties of the new directory are copied, such as compression and encryption settings

### **File functions**

Once a file handle is open, there is some basic information about the file that can be queried.

```
GetFileSize() - Get File size as a return value
```

`GetFileSize` returns the low 32-bit of the file size as its return value, and the high 32-bit value in the `lpFileSizeHigh` parameter, if specified. If `lpFileSizeHigh` is NULL, the high 32-bit value is not returned. The function returns INVALID_FILE_SIZE in case of an error, defined as 0xffffffff

low 32-bit used if the file size is < 4 GB (file size is less than 4 GB (2³²)) , example:

| File size | nFileSizeHigh | nFileSizeLow |
| --------- | ------------- | ------------ |
| 1 KB      | 0             | 1024         |
| 3 GB      | 0             | 3221225472   |
| 5 GB      | 1             | 1073741824   |
So for files under 4 GB, the **low 32 bits alone represent the full size**.
if file size ≥ 4 GB `HighPart` stores the overflow beyond 32 bits

The maximum file size in practice is much more limited than the theoretical 2 to the 64th power (16 EB) bytes

`ULARGE_INTEGER` struct provided in `WinNT.h` can be used to save a lower and upper bit values
```c
ULARGE_INTEGER size;  
size.LowPart = nFileSizeLow;  
size.HighPart = nFileSizeHigh;  
  
QWORD fileSize = size.QuadPart;
```

```
GetCompressedFileSize() - Get size of a compressed file, it takes a name rather than a handle
```

```
GetFileTime() - Gets basic information about a file relates to its creation, modification and access times
```

```
GetFileAttributes() - Returns file attributes of a file
```

![[Pasted image 20260316210218.png]]

```
GetFileAttributesEx() - just like previous one but returns a WIN32_FILE_ATTRIBUTE_DATA struct containing the info
```
![[Pasted image 20260316210330.png]]

```
GetFileInformationByHandle() - Gets more information of file using handle including size , creating & mod times , attributes (struct BY_HANDLE_FILE_INFORMATION)
```

```
SetFileAttributes() - Used to set attributes to file

The attributes that can be set by this function are: FILE_ATTRIBUTE_ARCHIVE, FILE_ATTRIBUTE_HIDDEN, FILE_ATTRIBUTE_NORMAL, FILE_ATTRIBUTE_NOT_CONTENT_INDEXED, FILE_ATTRIBUTE_OFFLINE, FILE_ATTRIBUTE_READONLY, FILE_ATTRIBUTE_SYSTEM, FILE_ATTRIBUTE_TEMPORARY
```

```
SetFileTime() - set file creation and access times
```

```
SetFileInformationByHandle()
```

## ==**Synchronous I/O**==

The functions that are synchronous, which means the calling thread is now blocked (goes into a wait state) until the operation is complete and the data has been transferred.

When calling `CreateFile` and not specifying `FILE_FLAG_OVERLAPPED` as part of the `dwFlagsAndAttributes` parameter, the file object is created for synchronous I/O only. This is the simplest to work with, so we’ll tackle synchronous I/O first.

The main functions to perform I/O (synchronous or Asynchronous) are `ReadFile` and `WriteFile`, which work with any file object (not necessarily pointing to a file system file)

```
ReadFile() - Read data from file and store in buffer
```

```
WriteFile() - write data from buffer into file
```

The last parameter, `lpOverlapped`, is required to be non-NULL for asynchronous operations, but for Synchronous I/O it should be NULL.

Each file object opened for synchronous access maintains an internal file pointer, that is automatically advanced with each I/O operation. For example, if a file is opened and a read operation of 10 bytes is performed, the file pointer advances by 10 bytes after the operation completes. If another read for 10 bytes is issued, it reads bytes 10 to 19, and the file pointer advances to position 20 in the file.

For sequential reads and writes, that’s great. In some cases, however, you may want to jump forward or back and read/write from a different position. This can be accomplished with one of the following functions:

```
SetFilePointer() - Sets the file pointer to a specific position
```

The functions move the internal file pointer to the desired position. `SetFilePointerEx` is easier to use, since it allows a full 64-bit offset to be provided in the `liDistanceToMove` parameter. `SetFilePointer` accepts the low 32-bit of the offset in `lDistanceToMove` and optionally the high 32-bit in `lpDistanceToMoveHigh`. Both functions attempt to return the previous file pointer

The offset to move, however, is not necessarily the offset from the start of the file. The last parameter, `dwMoveMethod`, indicates how to interpret the provided offset:
	• FILE_BEGIN (0)- from the beginning of the file
	• FILE_CURRENT (1) - from the current file position
	• FILE_END (2) - from the end of the file

```
SetEndOfFile() - sets the end of file to the place where the file pointer is at
```

## ==**Asynchronous I/O**==

The Windows I/O system is asynchronous in nature. Once a device driver issues a request to its controlled hardware (such a disk drive), the driver does not need to wait for the operation to complete. Instead, it marks the request packet as “pending” and returns to its caller. The thread is now free to perform other operations while the I/O is in progress. 

After some time the I/O operation completes by the hardware device. The device issues a hardware interrupt that causes a driver-supplied callback to run and complete the pended request.

It is inconvenient for threads to keep waiting for I/O operations to complete . Asynchronous I/O provides a solution, where a thread initiates a request, and then goes back to serve the next request, and so on, since I/O operations operate concurrently while the CPU executes other code. The only wrinkle in this simplistic model is how a thread is notified of an I/O operation completion.

One of the consequences of a file opened for asynchronous access is that there is no file pointer anymore. This means every operation must somehow provide an offset from the start of the file to perform the operation

We provide an OVERLAPPED structure explained below:
![[Pasted image 20260317040308.png]]

Most important members are the `Offset` , `hEvent` , `Pointer`  

Technically, `Internal` holds the I/O operation’s error code. For an asynchronous operation in progress, it holds `STATUS_PENDING`, which is the kernel equivalent of STILL_ACTIVE (0x103). In fact, Windows defines a macro, `HasOverlappedIoCompleted` that takes `advantage` of this fact.

In an asynchronous operation, the `ReadFile` or `WriteFile` calls normally return FALSE, since the operation is not yet complete, it just started. If the functions return FALSE, then calling `GetLastError` returns `ERROR_IO_PENDING`, it means everything is working as planned

Once the operation completes, how can you tell how many bytes were transferred? There are multiple Ways:
- Thread can wait for the `hEvent` object
- The thread can wait through `GetOverlappedResult`

```
GetOverlappedResult() - accepts the file handle and the OVERLAPPED structure for the particular operation you’re interested in (you can initiate several operations from the same file handle, each having a different OVERLAPPED instance). lpNumberOfBytesTransferred returns the number of bytes actually transferred.
```
The final parameter, `bWait`, specifies whether to wait for the operation to complete (TRUE) before reporting the result or not.

```
GetOverlappedResultEx() - The function allows setting a timeout limit for waiting (dwMilliseconds), and also the ability to wait in an alertable state (bAlertable)
```

![[Pasted image 20260317062855.png]]

```
WriteFileEx - ReadFileEx() - The functions are identical to their non-Ex counterparts, except for an additional argument, which is a function pointer
```

```c
typedef VOID (WINAPI *LPOVERLAPPED_COMPLETION_ROUTINE)(
_In_ DWORD dwErrorCode,
_In_ DWORD dwNumberOfBytesTransfered,
_Inout_ LPOVERLAPPED lpOverlapped
);
```
the callback is wrapped in an Asynchronous Procedure Call (APC) and queued to the original thread that called `ReadFileEx` or `WriteFileEx`. This means this particular thread is the only one which can invoke the callback by getting into an `alertable` state,

```
QueueUserAPC() - Thefunction queues an APCtothetargetthread represented by thehThread parameter, that must have the THREAD_SET_CONTEXT access mask. pfnAPC is a function pointer
```

```c
typedef VOID (WINAPI *PAPCFUNC)(_In_ ULONG_PTR Parameter);
```

This is still an APC, and so the target thread must enter an alertable state if the APC callback is to be executed.

## ==**I/O Completion Ports**==

I/O completion ports deserve their own major section, since they are useful not just for handling asynchronous I/O. We met them briefly when discussing jobs in chapter 4- a job can be associated with an I/O completion port, to receive notifications related to the job.

The I/O completion port’s purpose is to allow processing of completed I/O operations by worker threads, where “worker” here can mean any threads that are bound to the completion port.

An I/O completion port is associated with a file object (could be more than one). It encapsulates a queue of requests, and a list of threads that can serve these requests once completed. Whenever an asynchronous operation completes, one of the threads that waits on the completion ports should wake up and handle the completion, possibly initiating the next request

```
CreateIoCompletionPort() - create an I/O completion port object and associating it with one or more file handles.
```
The function can perform two distinct operations, possibly combining the two. It can do any of the following:
	• Create an I/O completion port not associated with any file objects. 
	• Associate an existing completion port to a file object.
	• Combine the above two operations in a single call.

Function is 1st called with Null , 0 , INVALID_HANDLE_VALUE at the beginning to create an Empty IOCP

Objects created by other functions such as Sockets can also be associated with an I/O completion port.

a completion port is always local to the process that created it. Technically, duplicating such a handle to another process succeeds, but the new handle is unusable.

Once a I/O completion port object is created, it can be associated with one or more file objects (handles).For each file handle, a completion key is specified, which is application defined.

_CompletionKey_ parameter help your application track which I/O operations have completed. This value is not used by **`CreateIoCompletionPort`** for functional control; rather, it is attached to the file handle specified in the `FileHandle` parameter at the time of association with an I/O completion port.

```
GetQueuedCompletionStatus() - The call puts the thread into a wait state until an asynchronous I/O operation initiated with one of the file objects associated with the completion port completes, or the timeout elapsed.
```

Typically `dwMilliseconds` is set to INFINITE, meaning the thread has nothing to do until an I/O operation completes. If the operation that wakes up the thread completed successfully, `GetQueuedCompletionStatus` returns TRUE and the out parameters are filled with the number of bytes transferred, the completion key originally associated with the file handle and the OVERLAPPED structure pointer that was used for the request.

The I/O completion port will not allow more than the maximum threads specified at the port’s creation to succeed the call at the same time.[^1]

Once a thread calls `GetQueuedCompletionStatus` the first time, it becomes bound to the completion port, until the thread exits, the completion port is closed, or the thread calls `GetQueuedCompletionStatus` on a different completion port.

It is possible to manually post a completion packet to an I/O completion port which makes these objects more generic, and not really just about I/O. This is exactly how the notifications work for a job object.

```
PostQueuedCompletionStatus() - post a completion packet to an I/O completion port
```

Remember: **IOCP lets many I/O operations share a small number of threads, instead of assigning a thread per operation.**
![[Pasted image 20260317070838.png]]

Example:
```
1. Issue async recv on many sockets  
2. No threads blocked  
3. OS completes I/O in background  (async)
4. Completion is queued  
5. One of a few threads wakes up and handles it
```

Once `GetQueuedCompletionStatus` finishes waiting, a worker thread dequeues a completed I/O packet from the IOCP queue, and the code following `GetQueuedCompletionStatus` executes as the handler for that completion (i.e., a manual “callback”).

### What an IRP really is

**IRP (I/O Request Packet)** = a **structure in memory** created by the Windows I/O Manager.

It describes:

- what operation (read/write)
- buffer
- size
- file/device
- status

Think of it like:
“do this I/O operation”

1. I/O Manager creates IRP  
2. IRP is passed to drivers (CPU runs driver code)  
3. Driver sends request to hardware  
4. Driver marks IRP pending and returns  
5. Hardware completes operation → triggers interrupt  
6. Driver handles interrupt (ISR/DPC)  
7. Driver completes IRP (`IoCompleteRequest`)  
8. Kernel queues a COMPLETION PACKET to IOCP  
9. Worker thread wakes up (`GetQueuedCompletionStatus`)  
10. Thread processes result (your “callback” code)

---
# Footer

[^1]: However, if a thread for which `GetQueuedCompletionStatus` succeeded, and that thread, while processing the completion operation enters a wait state for whatever reason (`SuspendThread`, `WaitForSingleObject`. etc.), the completion port will allow another thread to have its `GetQueuedCompletionStatus` call end its wait.
