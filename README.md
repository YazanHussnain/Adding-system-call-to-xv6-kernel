# Adding-system-call-to-xv6-kernel
Adding getreadcount system call to xv6. This system call return the number of times the read system call, has been called by user processes since the time that kernel was booted.

## Adding variable to Per-Process State structure
As we want to keep record of read() system call per process. Therefore, first file that we modify is ```proc.h```. In this file you can see the Per-Process structure, add integer to the structure which contain the count value and return, when ```getreadcount``` system call is made.

```ruby
// Per-process state
struct proc {
  uint sz;                     // Size of process memory (bytes)
  pde_t* pgdir;                // Page table
  char *kstack;                // Bottom of kernel stack for this process
  enum procstate state;        // Process state
  int pid;                     // Process ID
  // ===============================================================
  int readcnt;                 // sys_read count - added -
  // ===============================================================
  struct proc *parent;         // Parent process
  struct trapframe *tf;        // Trap frame for current syscall
  struct context *context;     // swtch() here to run process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
};
```

We declare the variable successfully. Now, we have to initialize it to zero, because we increment it when a '''read''' syscall is made. The next file that need modification is ```proc.c```. In this file you can see the function ```allocproc``` with return data type ```static struct proc*```. Below found initialize the variable which you declare in ```proc.h``` file as ```p->readcnt```

## Add getreadcount system call function
The file which need changes is ```sysproc.c```. In this, file you find the system call ```getpid```. The same implementation is done for ```getereadcount``` because we declare our varibale in per-process struct.
```ruby
int
sys_getreadcount(void)
{
    return myproc()->yourvariable;
}
```

## Add system call number
Next thing to add into the kernel for getreadcount system call is the system call number. In ```syscall.h``` file there are macros defined, those are simple integers add this line at the end ```#define SYS_getreadcount 22```

## Implement getreadcount system call
Next change that needed, is to make getreadcount to work. To do this you have to change the file ```syscall.c```. this file has a function named syscall with void as its argument. you will find the ```num``` variable inside that function, which is simply the system call number. In this function you simply need to compare num with SYS_read, which is the system call number of read system call. If the conditoion is true, increment in the counter. You can add a global variable, may named ```readcount```, in the file. 

Some more line of code need to be added. You also need to compare ```num``` with ```SYS_getreadcount```, in this ```if``` you assign the counter to the variable which you declare in per-process struct. Also add spinlock to work with multiple CPUs. Remember, initlize the struct spinlock instance globally to work with multiple CPUs.
When you initialize a spinlock structure locally, you define the spinlock structure inside a function or a block of code, and it only exists and is accessible within that scope. Once the function or block of code is finished, the spinlock structure is automatically destroyed. This type of spinlock is useful when you only need to protect a limited section of code and want to ensure that the spinlock is only used by that code.

On the other hand, when you initialize a spinlock structure globally, you define the spinlock structure outside of any function or block of code, and it is accessible from anywhere in your code. A global spinlock structure exists for the lifetime of the program and can be used to protect shared resources that are accessed by multiple parts of your code. This type of spinlock is useful when you need to protect resources that are shared by multiple threads or functions and need to ensure that only one thread or function can access the resource at a time.

```ruby
struct spinlock lock;             // added
```

```ruby
void
syscall(void)
{
  int num;
  struct proc *curproc = myproc();
  
  num = curproc->tf->eax;
  // ====================added======================
  if(num == SYS_read)
  {
    acquire(&lock);
    globalCountVaribale;
    release(&lock);
  }
  if(num == SYS_getreadcount)
  {
    curproc->yourvaribale = globalCountVaribale;
  }
  //====================added======================
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    curproc->tf->eax = syscalls[num]();
  } else {
    cprintf("%d %s: unknown sys call %d\n",
            curproc->pid, curproc->name, num);
    curproc->tf->eax = -1;
  }
}

```

To make sys_getreadcount accessible to other files, write ```extern``` with it prototype. Also add in the nice structure of system call array. This array tells us that the function should have return data type ```int``` and takes no arguments and our system call also need to be added into it.

``` [SYS_getreadcount]   sys_getreadcount ```

``` extern int sys_getreadcount(void) ```

## Changes in ```user.h``` and ```usys.S``` file
You now have to make edits to two small files that will provide the interface for your user program to access the system call. Open the file named ```usys.S``` and add the following line to the end. 

```SYSCALL(getreadcount)```

Then, open the file user.h and add the specified line. 

```int getreadcount(void)```

The function in ```user.h``` will be called by the user program, however, it does not have an actual implementation in the system. Instead, calling this function from the user program will be translated to system call number 22, represented by the SYS_getreadcount preprocessor directive
