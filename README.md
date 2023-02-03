# Adding-system-call-to-xv6-kernel
Adding getreadcount system call to xv6. This system call return the number of times the read system call, has been called by user processes since the time that kernel was booted.

## Adding variable to Per-Process State structure
As we want to keep record of read() system call per process. Therefore, first file that we modify is ```proc.h```. In this file you can see the Per-Process structure, add integer to the structure which contain the count value and return, when ```getreadcount``` system call is made.


We declare the variable successfully. Now, we have to initialize it to zero, because we increment it when a read syscall is made. The next file that need modification is ```proc.c```. In this file you can see the function ```allocproc``` with return data type ```static struct proc*```. Below found initialize the variable which you declare in ```proc.h``` file as ```p->yourvariable```

## Add getreadcount system call
The file which need changes is ```sysproc.c```. In this, file you find the system call ```getpid```. The same implementation is done for ```getereadcount``` because we declare our varibale in per-process struct.
```ruby
int
sys_getreadcount(void)
{
    return myproc()->yourvariable;
}
```

Add remaining decription later
