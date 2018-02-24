## What is a Process?
A process is a running instance of a program (binary source code).  
Each process is allocated a system wide unique process ID, which is used to identify the process.  

The process Id's are allocated in increments of 1, till the time the max value of 32,767 is reached. Once this value is reached the PID counter is reset to 300 and the available ids are reused.  

> File: */proc/sys/kernel/pid_max* contains the limit of 32,767, which can be increased.  

> The *init* process has PID of 1. 

Process Ids below 300 are generally used by system specific processes which generally runs for long durations. Thus, the
PID counter is reset to 300 and not 1. As most of the processes below PID 300 never dies out, leaving very rare chance of unused prcess ID below 300.  


## Concept of Virtual Memory:
![Virtual Memory Layout](https://lh4.googleusercontent.com/9CHLYizZaXekDRoskNEHrWHRY4_SROrCo6AsMR0BCuOzFDjGikdf_3IzIA13UuSwLrHrbCbacoU_BngXxzXo=w1436-h736)

Virtual memory model was introduced to overcome the limitations of physical memory i.e RAM.  
In this model each process is allocated a virtual memory space.  

The process only talks to virtual memory space and its the role of CPU
to translate the virtual memory addresses into physical memory addresses.
This transalation is done via the help of **Page Tables**. Like virtual memory, each process has its own Page Table too.

Both *physical* and *virtual* memory are divided into small memory chunks called **PAGES**. Each PAGE in physical memory is allocated a unique identifier called **Page Frame Number (PFN)**.
PAGES in virual memory are allocated **Virtual Page Frame Number (VPFN)**.

## Demand Paging & Page Fault:

***Demand Paging*** is a technique that helps in using the **physical memory** efficiently.  
It says that, instead of loading all the virtual pages in physical memory load only the virtual pages which are required by the program to function.  

When the CPU tries to access a *virtual page* (VPFN), which is not present in the physical memory and thus does not have an associated PAGE TABLE entry (*Like, VPFN 2 for process X in our above figure*) an exception is thrown by CPU which is called ***PAGE FAULT***.  
     
At the time of page fault the execution of the process is halted by CPU, till the point Kernel loads the respective page in physical memory and makes the corresponding page table entry.  
After the page table entry is made, the execution of the process resumes from the point the memory exception was thrown.

As it can take considerable time to load a page from disk to physical memory, CPU is free to execute any other process during that time.

An example of this is, if we are running a database server which can be queried to fetch data, it does not makes sense to store all the fetched records in physical memory instead store only the first few records and insert more records when the cursor demands them.

## Swapping and Thrashing: 
   
When the physical memory is Full and a process requests a *new page* that is not already present in physical memory. The Kernel needs to make
   space for the new page by evicting a *less used page* from physical memory.
   
   If the page to be evicted already exists on disk and no changes have been made to it then it is simply evicted.

   However, if the page does not exists on disk or the page has been loaded from disk but have some local modifications than we can't
    simply evict the page because when the process will require this page again then we will not be able to provide it as the contents 
    would be lost, such a page is called "**Dirty Page**".

The Dirty Pages are stored in a separate area on disk called "**Swap Space**", So that when needed these pages can be pulled from swap area and loaded to physical memory.

The process of evicting old pages and loading new pages is called ***Swapping***. However,
If the Kernel is constantly evicting and loading new pages again and again its called ***Thrashing***.  

> We generally have page size of 4096 bytes = 4Kbyte.

> System Page Size configuration can be retrieved by running following bash command:  
    ```getconf PAGE_SIZE```  

## Memory Layout of a Process:

Each process is assigned a virtual memory space which is **isolated** from any other processes in the system.
This virtual space is divided into segments. Each segment keeps different type of data related to the process.

Following are the segments:
1. **Text Segment:** This segment stores the program source code in "machine executable format". Since, multiple processes may
   be running the same program this segment is generally shared among multiple processes of same program.
   Also, this segment is kept as Read-Only so that process cannot modify its source code while executing.

2. **Initialized Data Segment or Data Segment:** This is the place where GLOBAL and STATIC variables are stored.  
```ex: int primes[] = { 2, 3, 5, 7 };```

3. **UnInitialized Data Segment or BSS:** This place also keeps the GLOBAL and STATIC variables but the ones which are not initialized.  
```ex: char globBuf[65536];```   

4. **Stack:** This is a dynamically growing and shrinking segment containing *stack frames*. Each function is allocated one stack frame.  
A frame contains functions local variables, return values and arguments.

5. **Heap:** This is also a dynamicaly growing and shrinking area used to store variables.

![Process Memory Layout](https://doc-14-bg-docs.googleusercontent.com/docs/securesc/ocmr2p55n30qghcdh868gukr4b79rr7n/k5vqn5remrffdbj4l6clc1jkb10qbgn0/1519452000000/06891064801342442300/06891064801342442300/1IaL5mC-5m0OkSCu2oSvHUu0ggCFngoju)

Following is the code example for the same.

```
main(int argc, char *argv[])       /* Allocated in frame for main() */
{
  static int key = 9973;           /* Initialized data segment */
  static char mbuf[10240000];      /* Uninitialized data segment */
  char *p;                         /* Allocated in frame for main() */
  p = malloc(1024);                /* Points to memory in heap segment */
  doCalc(key);
  exit(EXIT_SUCCESS);
}
```

## Creating New/Child Processes: 
A parent process can create new process by using the "fork()" system call. Such a process is called Child Process.
The child process is a exact copy of the parent process, this copy includes: Parents Stack, Heap, Text Segment, Initialized Data Segment.

## Terminating a Process:
A process can be terminated using the exit(int status) library function which wraps over _exit(int status) system call.
Generally, following convention is used, if "status" variable is 0, means the process has succesfully completed its task, A non Zero status illustrates that process encountered some error in its task.

When a process terminates, all the file descriptors opened by it are closed. The virtual memory allocated to it is destroyed and any child processes
now have "init" process as their parent process. 

**Exit handlers:** the "exit()" library function allows us to define two functions "on_exit()" and "at_exit()". These are used as pre-exit hooks 
  for process. So, incase some additional cleanup task is to be done before process terminates these two functions are a go to.
  However, these functions donot get called if the process skips exit() call and terminates by calling _exit() system call directly. Or the process
  is terminated abnormally by a SIGNAL.

### "wait()" system call:
The wait() system call has two purposes:
 1. If none of the child of this process has not terminated it will suspend the execution of the process, until one of the child terminates.
 2. The wait(&status) call accepts a status parameter, which is used to track the termination status of child.   

### "execve()" system call:

execve(pathname, argv, envp)
 
The execve() system call loads a new program, specified with 'pathname' with the argument list supplied in "argv" and the environment
list provided in "envp" into process virtual memory space. 
In Such a case, the existing programs heap, stack, text and other segments are removed from virtual memory space and new program's
data segments, stack, heap etc are loaded.

The below figure illustrates, How a process running program "A" executes another program "B" using child process technique.

Fig:24-1 page number 515.

## File Sharing between Parent and Child:

When a fork() is performed, the child receives duplicates of all the parent's file descriptors.
These duplicates are such that even the file descriptor attributes are shared, 
if either of the process changes the offset of read pointer it will be reflected in other process.
@todo: Need a code example for this.

Sharing the file descriptor in such a manner ensures that, when both the process are writing to file they don't overwrite each others data.
However, note that the data can be intermingled.

If this is not the desired behaviour, then the application should be designed in such a manner that after the fork() parent and child uses
different file descriptors. This process is illustrated in below figure: (fig page 520)

## Race Conditions after fork():

Once a fork is performed, whose code will be executed first? Is it child process code that will be executed first or Is it parent process
which will execute first?
Ans: Its indeterminent that which one will run first, on multi processor systems both can run parallely. However, most of the time its the child
that executes first.
This is controlled via following setting: "/proc/sys/kernel/sched_child_runs_first"


## Monitoring childs:

wait(), waitpid() and other similar calls.

## Orphan processes and Zombies:

If the parent process terminates before the Child process has completed its job and terminated. Then the Child process is called
an Orphaned process and is adopted via the "init" process in process hierarchy.

Now, Consider the case, when the Child process has already terminated way before the parent process calls a wait() or waitpid() function 
to retrieve the termination status of the Child. As we already know that upon process termination all the related information is also destroyed, so how
the kernel will be able to provide the termination status in such a case?
To handle such cases, kernel destroys everything related to child process but retains an entry in process table for the child process. 
This entry contains: processId, termination status and process resource usage statistics.
Such a Child Process, which is already terminated but is waiting to be called by parent's wait() or waitpid() function is called a Zombie process. As
its actually dead but still have some sort of life in it.

One thing to note here is that, a Zombie process is very much like zombies in movies. i.e a Zombie process cannot even be killed via SIGKILL signal (silver bullet).
The only way to kill a zombie process is, when te parent calls wait() or waitpid() family function on it.
Incase, the parent process terminates before calling a wait() on a Zombie process, the zombie is adopted via "init" process which automatically calls
wait() on the zombie and kills it.

NOTE: Its mandatory to handle zombie processes when you are dealing with child processes in your application. Not doing so, will have the effect
that the entry from process table for the child process will not be removed and can easily eat up the process table. Thus, preventing you to create
more processes.

Handling Zombies: Zombies can be handled by using wait() family functions or by using SIGCHLD Signal. (Not diving in this as of now)

## Threads:

Like processes, Threads are a mechanism that allows an application to perform multiple tasks concurrently.

A single process can have multiple threads running in it. All of these threads can execute parallely the same program code. 
They share all the memory segments with each other such as: Initialized Data, Uninitialized data, Text Segment and Heap. But have independent Stack data.
See fig: pg 618

NOTE: By default each process has one thread. This "one" thread is called as Main Thread.

A new thread can be created by calling the "clone()" system call.

Notion of threads provide us with the ability to choose between "MutiThreaded" and "MultiProcess" application approach. 


## MultiThreaded vs Multiprocess:

1. Information Sharing: Sharing of information between processes is bit difficult since parent and child does not share memory. In order to do so
     we must use some sort of IPC.
     Sharing information b/w threads is easy and fast. Its just a matter of copying data into shared memory segment(global variable or heap).

2. Creating Childs: Creating a new child process (using fork()) is much costlier than creating new threads (using clone()). Its almost 10 times
      more costlier.

3. Coding Complexity: It is comparitively easier to create multiprocess applications than multi threaded applications. As if not handled properly, one thing 
      going wrong in any one of the threads can affect all the threads within the process since they all share same memory segments.

## System and Process Related Info: 

Most of the System and Process related limits and settings are stored in /proc directory.
/proc directory does not exists on the filesystem and is created by kernel on the fly, thus its a virtual file system. However, you can read 
and write to it using basic file IO commands.

Some important files and directories to remember are:

- /proc/PID/cmdline -> shows command line arguments sent to process, shows executable file path as well.
- /proc/PID/environ -> environment variables when process was spawned.
- /proc/PID/status -> shows general info about process.
- /proc/PID/cwd -> symlink to current working directory.
- /proc/PID/exe -> symlink to file being executed.
- /proc/PID/fd -> symlinks to files, sockets, pipes opened by this process.
- /proc/PID/task -> contains one subdirectory for each process.
- /proc/sys -> contains system related limits and settings.

NOTE: In order to edit/read contents of a file inside /proc directory do not use any editor it can jumble up things. Instead 
     use following command structure:

     # echo 100000 > /proc/sys/kernel/pid_max
     # cat /proc/sys/kernel/pid_max
     100000 







