# Adding-system-call-to-xv6-kernel
Adding getreadcount system call to xv6. This system call return the number of times the read system call, has been called by user processes since the time that kernel was booted.

## Adding variable to Per-Process State structure
As we want to keep record of read() system call per process. Therefore, first file that we modify is ```proc.h```. In this file you can see the Per-Process structure, add integer to the structure which contain the count value and return, when ```getreadcount``` system call is made.


We declare the variable successfully. Now, we have to initialize it to zero, because we increment it when a read syscall is made. The next file that need modification is ```proc.c```. In this file you can see the function ```allocproc``` with return data type ```static struct proc*```. Below found initialize the variable which you declare in ```proc.h``` file as ```p->yourvariable```

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
Next change that needed, is to make getreadcount to work. To do this you hav eto change the file ```syscall.c```. this file has a function named syscall with void as its argument. you will find the ```num``` variable inside that function, which is simply the system call number. In this function you simply need to compare num with SYS_read, which is the system call number of read system call. If the conditoion is true, increment in the counter. You can add a global variable, may named ```count```, in the file. 

Some more line of code need to be added. You also need to compare ```num``` with ```SYS_getreadcount```, in this ```if``` you assign the counter to the variable which you declare in per-process struct. Also add spinlock to work with multiple threads.

```ruby
void
syscall(void)
{
  int num;
  struct proc *curproc = myproc();
  struct spinlock lock;             // added
  initlock(&lock, "readCnt");       // added
  
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

Add a line like other system call wiht extern keyword and also add in the nice structure of system call array.

``` [SYS_getreadcount]   sys_getreadcount ```

``` extern int sys_getreadcount(void) ```

## Changes in ```user.h``` and ```usys.S``` file
Add this line to user.h file

```int getreadcount(void)```

Add this line to usys.S file

```SYSCALL(getreadcount)```
