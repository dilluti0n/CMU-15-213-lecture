# 6. Exceptional Control Flow
> lecture source: [14-ecf-procs.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/14-ecf-procs.pdf) [15-ecf-signals.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/15-ecf-signals.pdf)

### **Control flow**

A program is sequence of instrucions (Say I{n}). Also, their respective adresses form a incrementing sequence a{n}. The instruction I{k} loaded at some address a{i} will move the program counter to a{j} after execution, and the CPU will execute the instruction I{k+1} loaded at that address. This flow of programs is called control flow. If j = i+1, (i.e. Ik moves program counter to next adress) we say the flow through Ik is _smooth_; otherwise, it is _abrupt_.

- smooth - add, ...
- abrupt - jmp, call, ret

### **Exceptional control flow**

If you don't need a kernel, i.e. if your program can execute directly on hardware without the need for an intermediary, exceptional control flow becomes unnecessary because it is only required to allow the kernel to intervene in a currently running program.

It is part of abrupt control flow, however does not control by the program's instructions. Instead, control is switched to kernal code and handled by it.

- Exceptions - triggered by `something`, handled by `handler`, which is kernal routine loaded in boot-time.
  1. Interrupts - triggered by h/w events. (like keyboard input)
  2. Traps - triggered by certain instruction on user code, `syscall`
  3. Faults - triggered by error conditions that a handler might able to correct.
  4. Aborts - triggered by unrecoverable fatal errors. (like detatch the RAM or something ...)

- Context switch - Process scheduling
  - `process` is an abstraction that allows to kernal controls bunch of diffrent executables.

- Signals - How processes communicate with each other. (like Ctrl-C on shell)

## $6.1. Exceptions

- Transfer of control to the OS kernel in response to some _event_
- **Events** - Divide by 0, arithmetic overflow, page fault, I/O request, typing Ctrl-C ...
- **Exception Tables**
  - Each type of event has a unique exception number k.
  - k = index into exception table (a.k.a. inturrupt vector)
  - Handler k is called each time exception k occurs.
- Exception Table and handlers are loaded to memory on boot-time.
- When exception occurs on instructions Icurr, it returns to Icurr or Inext or does not returns(abort).

### Inturrupts

- Caused by events external to the processer. (e.g. keyboard pressed, Timer interrupt, ...)

1. Event occured (cpu's inturrupt pin goes high)
2. Icurr executed (Note; even if an event occurs while Icurr running, an exception occurs _after_ Icurr completes execution.)
3. Read exception number from system bus.
4. Call to inturrupt handler. (When this, adress to Inext pushed to kernel stack)
5. After handler executed, it returns to Inext.

### Traps (syscall)

- Caused by execution of instruction `syscall`.
- Kernal provide procedure-like services such as 
  - `read` - read a file
  - `fork` - create new process
  - `execve` - load a new program to current process
  - `exit` - terminate the current process
- For programmer's perspective, it is identical to regular call
- However they are executed on `kernel mode`, not `user mode`
- Returns control to next instruction of `syscall`.

```asm
mov $0x2, %eax # syscall ID 2 is open
syscall        # Return value in %rax
cmp $0xfffffffffffff001,%rax
```

### Faults

- Result from error conditions that a handler might able to correct.
  - _able_ to correct - Returns to faulting execution and re-execute it!
  - _disable_         - Abort
  
**Example**

- Page Fault (recoverable)
  1. The process accesses not-loaded adress. (or instruction itself has been not-loaded yet)
  2. Page fault handler loads current page to memory from disk.
  3. The handler returns to Icurr & re-execute it!
  - More explaination on VM part..
- Segmentation Fault (Abort)
  1. The process accesses to illigal adress
  2. Segmentation fault handler delivers SIGSEGV to process.
  3. The Process terminated (or Signal handler for SIGSEGV executed, if it is planted for that process..)

### Aborts

- Result from unrecoverable fatal errors.
- always abort
- EX) Memory bits are corrupted, ...

## $6.2. Process and context switch

- _process_ - instance of running program.
- Key abstractions provided by _process_
  1. Logical control flow - _context switching_
    - Each program seems to have exclusive use of the CPU
  2. Private adress space - _virtual memory_
    - Each program appears to have exclusive access to the main memory
- Two processes run _concurrently_ if their flows overlap in time (Otherwise, they are _sequential_)

**Implements**
For process control. (See [5_ShellLab/shlab-handout/tsh.c](https://github.com/dilluti0n/CMU-15213-lab-sol/blob/master/5_ShellLab/shlab-handout/tsh.c) also.)

- Process states
  - Running - Either executing or chosen to execute by kernel.
  - Stopped - The execution is _suspended_ and will not be scheduled until further notice(by signals).
  - Terminated - Stoped permanently.
  
- Process become **terminated** for one of three reasons:
  1. Receiving a signal whose default action is to terminate.
  2. Returning from the `main` routine
    - The return value become the exit status.
  3. Calling the `exit` function
    - `void exit(int status)`
      - Terminates with an exit status of `status`
      - Normal return status is 0, nonzero on error.
      - Called once, never returns.
      
- _Parent_ process creates a new running _child_ process by calling `fork`
  - `int fork(void)`
    - Returns 0 for child process, child's PID to parent process.
    - Called once, returns twice.
    - Child executes concurrently with its parent. - cannot predict order.
    - Duplicate but separate address space
    - Shared open files
  - **Example**
```c
int main()
{
    pid_t pid;
    int x = 1;
    
    if((pid = fork()) == -1) {
        perror("fork: ");
        exit(1);
    } else if (pid == 0) { /* Child */
        printf("child: x=%d\n", ++x);
        exit(0);
    }
    
    printf("parent: x=%d\n", --x);
    exit(0);
}
```
Result
```
linux> ./fork
parent: x=0
child: x=2
```
  - track the unpredictable cocurrent execution with _process graph_

- _Reaping_ child processes
  - zombie process - A process that has terminated but still consumes system resources.
    - Exit status, various OS tables, ...
    - These system resources are freed by _Reaping_ it from its parent.
  - _Reaping_
    - Performed by parent calling syscalls. (`wait`, `waitpid`)
    - Parent given exit status information.
    - Kernal then deletes zombie child 
(child process terminated -> reap by its parent -> kernal frees system resources using by child)
    - If any parent terminates without reaping a child, then the orphaned child will reaped by `init` process.
    - Only need explicit reaping in long-running processes.
      - shells, servers, ...

  - `int wait(int *child_status)`
    - Suspends current process until one of its children terminates. (and reaps that terminated children!)
    - Return `pid` of the child process that terminated
    - If `child_status` is non-`NULL`, then the integer it points to will be set to a value that indicates reason the child terminated and the exit status. (see `wait(2)`)
    
  - `pid_t waitpid(pid_t pid, int &status, int options)`
    - `wait` with various options rather than just suspends current process until _one_ of its children.

- _Loading_ and _Running_ Programs
  - `int execve(char *filename, char *argv[], char *envp[])`
    - Loades executable `filename` (ELF or script with `#!interpreter`) in the current process
    - ...with argument list `argv` (passe to loaded program)
    - ...and environment variable list `envp`
      - "name=value" strings (e.g. `PATH=/bin:/usr/local/bin`)
      - gloval variable `char **environ` on libc.
      - see `getenv(3)`, `putenv(3)`, `printenv(3)`.
