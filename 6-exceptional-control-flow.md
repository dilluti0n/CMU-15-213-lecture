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
2. Icurr executed (Note, even if an event occurs while Icurr running, an exception occurs _after_ Icurr completes execution.)
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
