
## ==**Thread Pools**==

This chapter talks about Thread pools, what are thread pools , what's the difference between them and normal threads, when to use them ,ETC .. 

In the last few chapters, we saw how to create and manage threads. Although this works, there are cases where it’s overkill. Sometimes we need to perform some bounded operation on a different thread, but always creating a new thread has its overhead. Threads are not free: the kernel has its structures that manage the information of a thread, a thread has usermode and kernel-mode stacks, and the creation of a thread itself takes time. If the thread is expected to be relatively short-lived, the extra overhead becomes significant.

Thread Pools are a bunch of threads that are managed by the windows kernel , each process has a thread pool that it can use even if it only has one thread, and you can see that when looking at the process in Process Explorer, that it has several threads, some of them starting with the function `ntdll!TppWorkerThread`. This is the starting function for a thread-pool thread. (“Tpp” is short for Thread Pool Private and the kernel responsible for thread pools is called `TpWorkerFactory`)

### ==**Thread Pool Work Callbacks**==

You can set a callback function for the thread pool to execute by multiple ways , simplest way is:

```
TrySubmitThreadpoolCallback - sets up a callback provided as the first parameter, to be called by the thread pool. The second parameter allows specifying a context value that is passed as-is to the callback function.
```

The callback function looks like this:
```c
typedef VOID (NTAPI *PTP_SIMPLE_CALLBACK)(
 _Inout_ PTP_CALLBACK_INSTANCE Instance,
 _Inout_opt_ PVOID Context
);
```

The first parameter of type  `PTP_CALLBACK_INSTANCE` is an opaque pointer than represents this callback instance.

Once submitted, the callback executes as soon as possible by a thread pool thread. There is no built-in way to cancel the request once submitted. There is also no direct way of knowing when the callback finished execution. It is possible, of course, to add our own mechanism, such as signaling an event object from the callback and waiting for it on a different thread. [^1]

Number of threads executing the load increases as the load also increases, there is no way to set the amount of threads to execute as its handled by the `TpWorkerFactory` kernel 

### ==**Controlling a Work Item**==

Sometimes you need more control. For example, you may want to know when the callback completes, or you may want to cancel the work item if some condition is satisfied. For these cases, you can create a thread pool work item explicitly

since `TrySubmitThreadpoolCallback` uses an internal work object, we are limited [^1]

**What are Work Objects?**  
A **work object** in the Windows API (specifically the Thread Pool API) is a structure that represents a callback task submitted to the Windows thread pool. It allows you to manage the execution of that callback, including:

- Cancellation
- Waiting for completion
- Re-submission
- Reference counting (lifetime management)
- Association with a callback environment
- Thread pool optimizations and reuse

it wraps and manages the callback execution inside the Windows thread pool system.

work object typically contains:
- Pointer to the callback function
- Pointer to user context State (submitted / running / canceled)
- Links so it can be queued
-  Callback environment reference

```
CreateThreadpoolWork - used to make a work object, The function looks similar to TrySubmitThreadpoolCallback, with two differences. The first is the return value, which is an opaque PTP_WORK pointer representing the work item, or NULL on failure. The second difference is the callback prototype
```

The callback prototype `CreateThreadpoolWork` takes is:
```c
typedef VOID (CALLBACK *PTP_WORK_CALLBACK)(
 _Inout_ PTP_CALLBACK_INSTANCE Instance,
 _Inout_opt_ PVOID Context,
 _Inout_ PTP_WORK Work
);
```

There is a third argument to the callback, which is the work item object returned originally from `CreateThreadpoolWork`.

After creating the callback and making the work object, we then proceed by submitting that callback for execution:
```
SubmitThreadpoolWork - submits a work object that has a callback function for execution
```

More than one call to `SubmitThreadpoolWork` is allowed using the same work object. The potential downside is that all these submissions use the same callback and the same context, as these can only be provided at work object creation.

We can then wait for work object using:
```
WaitForThreadpoolWorkCallbacks - waits for work object callback function to execute, the 2nd arg allows the function to wait for pending callback functions to execute or cancel pending callback
```

```
CloseThreadpoolWork - clean a work object
```

### ==**Thread Pool Wait Callbacks**==

thread pool thread can be waiting for multiple objects, submitted by the application and possibly other Windows APIs and libraries. Each such thread can wait for up to 64 objects at the same time using the familiar `WaitForMultipleObjects` and execute callback when wait is done

```
CreateThreadpoolWait - Creats a thread pool wait object, it takes same parameters as previous functions discussed, but the callback prototype is different, returns an opaque PTP_WAIT pointer that represents the wait object
```

```c
typedef VOID (NTAPI *PTP_WAIT_CALLBACK)(
 _Inout_ PTP_CALLBACK_INSTANCE Instance,
 _Inout_opt_ PVOID Context,
 _Inout_ PTP_WAIT Wait,
 _In_ TP_WAIT_RESULT WaitResult); // just a DWORD
```

`TP_WAIT_RESULT` specifies why the callback was invoked; that is, the callback is invoked when a wait operation for an object is over, so Wait result specifies why it was over. Possible values include `WAIT_OBJECT_0` to indicate the object was signaled and `WAIT_TIMEOUT` indicating the timeout expired without the object being signaled.

```
SetThreadpoolWait - invokes the thread pool wait object, the function accepts the handle to wait on, and the timeout argument specifies how long to wait.
```


```
SetThreadpoolWaitEx - similar to the previous function , but it returns a bool weather the wait has started or not
```

```
WaitForThreadpoolWaitCallbacks - waits for a wait object to finish
CloseThreadpoolWait - clean the wait object
```

### ==**Thread Pool Timer Callbacks**==

In chapter 8, we looked at a waitable timer kernel object that can be used to initiate operations when it expires, optionally periodically. However, initiating operations was not very convenient. It required some wait operation (which now we know can be accomplished with the thread pool), or running a callback as an APC on the thread that called `SetWaitableTimer`. The thread pool provides yet another service that calls a callback directly (from the pool) after a specified period elapses, and optionally periodically

The approach should be straight forward and clear without much details

```
CreateThreadpoolTimer - Creates a thread pool timer object
```

```c
typedef VOID (CALLBACK *PTP_TIMER_CALLBACK)(
 _Inout_ PTP_CALLBACK_INSTANCE Instance,
 _Inout_opt_ PVOID Context,
 _Inout_ PTP_TIMER Timer
 );
```

This is the callback of the function , we should be able to recognize the patterns.

```
SetThreadpoolTimer - invokes the thread pool timer object 
```
The `pftDueTime` parameter specifies the time for expiration, with the aforementioned format used with `SetWaitableTimer` and `SetThreadpoolWait`, event though it’s typed as FILETIME. If this parameter is NULL, it causes the timer object to stop queuing new expiration requests (but those already queued will be invoked when expiration is due). The `msPeriod` is the requested period, in milliseconds. If zero is specified, the timer is one-shot.

```
IsThreadpoolTimerSet - determine whether there is a timer set on the timer object
```

```
WaitForThreadpoolTimerCallbacks - wait for timer work object
CloseThreadpoolTimer - clean timer work object
```

### ==**Thread Pool Environment**==

There are 2 parameters that we didn't describe yet, lets start with `PTP_CALLBACK_INSTANCE` , which is used with several functions callable from the callback itself.

```
CallbackMayRunLong() , it takes a callback instance argument and provides a hint to the thread pool that this callback may be long running, so the thread pool should consider this thread not part of the thread pool
```

The next set of functions request the thread pool to perform a certain operation before the callback really ends and the thread can go back to the pool

```
SetEventWhenCallbackReturns()
ReleaseSemaphoreWhenCallbackReturns()
ReleaseMutexWhenCallbackReturns()
LeaveCriticalSectionWhenCallbackReturns()
FreeLibraryWhenCallbackReturns()
```

We wont talk about them since the name should explain everything

```
DisassociateCurrentThreadFromCallback() , this function tells the thread pool that this callback finished its important work, so other threads that may be waiting on functions such as WaitForThreadpoolWorkCallbacks to satisfy their wait, even though the callback is technically still executing
```

Now to the 2nd parameter which is the thread pool environment , Each of the thread pool object creation functions has a last parameter of type `PTP_CALLBACK_ENVIRON`, called a callback environment. Its purpose is to allow further customization when using thread pool functions.

```
InitializeThreadpoolEnvironment() - creates a new thread pool environment with clean state (the function currently zeroes all members of the new PTP_CALLBACK_ENVIRON struct except the Version field that is set to 3)
```

```
DestroyThreadpoolEnvironment() - destroy the environment
```

Once the environment is made , it can be used with a set of functions that customize the thread pool execution
![[Pasted image 20260304042251.png]]

*When making an environment don't forget to pass it to the Creating function of the work object you want*

![[Pasted image 20260304042535.png]]

When using one of the functions its better to seek the Microsoft docs, as we will not cover them here

### ==**Private Thread Pools**==

Thread pools automatically use the default process thread pool which cant be destroyed , With a callback environment, it’s possible to target callbacks to a different thread pool .

```
Private thread pool
|
|
Thread pool Environment -- > Work object -> Callback function
|
|
Customizations
Clean groups (explained later)
```

```
CreateThreadpool() - create a private thread pool object
```

```
SetThreadpoolCallbackPool() - sets an envrionment with a private thread pool object
```

With a thread pool in hand, functions are available to customize it to some extent. The most important functions are related to the minimum and the maximum number and size of the thread stack of threads in the pool

```
SetThreadpoolThreadMaximum()
SetThreadpoolThreadMinimum()
```

The default maximum number of threads is 512 and the minimum is 0. However, these numbers should not be relied upon, so it’s better to call the above functions to set appropriate values

```
SetThreadpoolStackInformation() , the function takes a struct that defines the size of commited and reserved pages (defined below)
```

```c
typedef struct _TP_POOL_STACK_INFORMATION {
  SIZE_T StackReserve;
  SIZE_T StackCommit;
 }TP_POOL_STACK_INFORMATION, *PTP_POOL_STACK_INFORMATION;
```

The names should explain everything.

The default sizes come from the PE header as described in chapter 5. That is, the default of the defaults is 4KB committed memory and 1MB reserved. This may be too much or too little, so feel free to set your own size

```
QueryThreadpoolStackInformation() , returns a `TP_POOL_STACK_INFORMATION` struct to query stack size 
```

```
CloseThreadpool() , close thread pool object
```

### ==**Cleanup Groups**==

A cleanup group keeps track of all callbacks associated with it so they can be closed with a single stroke, without the application having to keep track of all callbacks manually. Note that this applies to a private thread pool only, as the default thread pool cannot be destroyed.

A cleanup group is associated with a callback environment discussed in the previous section.

```
CreateThreadpoolCleanupGroup() - create a cleanup object
```

```
SetThreadpoolCallbackCleanupGroup() - links between environment with the cleanup object , it also takes a callback function thats executed when cleaning (its optional , can be NULL)
```

```c
typedef VOID (CALLBACK *PTP_CLEANUP_GROUP_CANCEL_CALLBACK)(
 _Inout_opt_ PVOID ObjectContext,
 _Inout_opt_ PVOID CleanupContext
);
```

```
CloseThreadpoolCleanupGroupMembers() - The function waits for all outstanding callbacks to complete. This saves you from calling the various close functions for the items created for this pool. If fCancelPendingCallbacks is TRUE, all callbacks that have not started yet are canceled, and the callback is executed
```

```
CloseThreadpoolCleanupGroup() - close the cleanup object
```

## Functions discussed in this chapter

```
TrySubmitThreadpoolCallback()

CreateThreadpoolWork()
SubmitThreadpoolWork()
WaitForThreadpoolWorkCallbacks()
CloseThreadpoolWork()

CreateThreadpoolWait()
SetThreadpoolWait/Ex()
WaitForThreadpoolWaitCallbacks()
CloseThreadpoolWait()

CreateThreadpoolTimer()
SetThreadpoolTimer()
IsThreadpoolTimerSet()
WaitForThreadpoolTimerCallbacks()
CloseThreadpoolTimer()

InitializeThreadpoolEnvironment()
DestroyThreadpoolEnvironment()

CreateThreadpool()
SetThreadpoolCallbackPool()
CloseThreadpool()

CreateThreadpoolCleanupGroup()
SetThreadpoolCallbackCleanupGroup()
CloseThreadpoolCleanupGroupMembers()
CloseThreadpoolCleanupGroup()
```

---
# Footer

[^1]: `TrySubmitThreadpoolCallback` uses an internal, ephemeral work object, You cannot: Cancel it Wait on it Re-submit it or Track its state
