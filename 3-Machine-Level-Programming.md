# 3. Machine Level Programming
> lecture source : [05-machine-basics.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/05-machine-basics.pdf), [06-machine-control.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/06-machine-control.pdf), [07-machine-procedures.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/07-machine-procedures.pdf)\
> lab assignment solution for Bomblab : [2_BombLab](https://github.com/codeAligned/CMU-15213-Lab/tree/master/2_BombLab)

## Basics

### \$3.1 C, assembly, machine code
- Definitions
  - Machine Code: The byte-level programs that a processor executes.
  - Assembly Code: A text representation of machine code.
  - Architecture: The part of a processor disign that one leads to understand or write assembly/machine code.(e.g. instruction set specification, registers.)
  - ISA(instruction set architecture): x86, IA32, x86-64(AMD64), ARM
  - Microarchitecture: Implementation of the architecture.(e.g. cache sizes and core frequency.)
- Compile process
  - `gcc -Og -S`: C(`.c`) to Asm(`.s`)`-S` with optimize`-Og`
  - Asm to object(`.o`) program(assembler: `gcc` or `as`)
  - object to executable(`no extension` or `.exe` or library: `.a`, `.dll`) program.(linker: `gcc` or `as`)
- Disassemble
  - `objdump -d sum`: prints binary `sum` to assembly on `stdout`.
  - `gdb(lldb) sum` and `disassemble sumstore(function name)` will also disassemble the particular function(`sumstore`) on binary sum.
  - there are legally restricted binaries to disassemble by license.

### \$3.2 Registers, operands, move for x86-64
- x86-64 Integer Registers

| 64-bit | 32-bit |
|:------:|:------:|
|%rax    |%eax    |
|%rbx    |%ebx    |
|%rcx    |%ecx    |
|%rdx    |%edx    |
|%rsi    |%esi    |
|%rdi    |%edi    |
|%rsp    |%esp    |
|%rbp    |%ebp    |
|%r(N)   |%r(N)d  |

(N): 8 ~ 15 integer.

  - there are 16 registers.
  - `%rsp` and `%esp` is special. They are called `stack pointer`, reserved for special use.
  - when you use right side of representation with `int`, it will only uses half of the bits of its register on processor.
  - left side of registers is used when you declare a variable like `long` or `char *`, `8-byte`.
- Moving data
  - `movq src dest`
  - Operand(`src`, `dest`) types
    - Immediate: constant integer data(litaral), prefixed with `$`.(e.g. `$0x400`, `$-533`.)
    - Register: one of 16 integer registers.(e.g. `%rax`, `%r13`.)
    - Memory: 8 consecutive bytes of memory at address given by address mode. (e.g. `(%rax)` represents memory location given of value stored in `%rax`, and there are more rules for "address mode".)
  - `movq Mem Mem` is illegal.
  - others are all legal.(`Imm` for `dest` is illegal. of course.)
- Memory Addressing Modes
  - `D(Rb,Ri,S)` is `Mem[Reg[Rb] + S*Reg[Ri] + D]`
    - `D`: constant "displacement"
    - `Rb`: Base register; any of 16 integer registers.
    - `Ri`: Index register; any, except for `%rsp`: stack pointer.
    - `S`: Scale; 1,2,4 or 8.
  - `(Rb,Ri)` is `Mem[Reg[Rb] + Reg[Ri]]`
  - `D(Rb,Ri)` is `Mem[Reg[Rb] + Reg[Ri] + D]`
  - `(Rb,Ri,S)` is `Mem[Reg[Rb] + S * Reg[Ri]]`
  - This is for array arithmetic.
- Example: `swap`

```c
void swap(long *xp, long *yp)
{
    long t0 = *xp;
    long t1 = *yp;
    *xp = t1;
    *yp = t0;
}
```

is compiled to

```asm
swap:
    movq    (%rdi), %rax     # %rdi: xp, %rax: t0
    movq    (%rsi), %rdx     # %rsi: yp, %rdx: t1
    movq    %rdx, (%rdi)     # %rdx: t1, %rdi: xp
    movq    %rax, (%rsi)     # %rax: t0, %rsi: yp
    rst                      # exit
```

  - Arguments of funtion will be continuously designated `%rdi`, `%rsi`, `%rdx`, `%rcx`, `%r8`, `%r9` (up to 6) by compiler.
  - In this case, it is `%rdi`: `xp` and `%rsi`: `yp`.
  - Since we declare the `8-byte` `long` variables, following registers are `64-bit` sets.

### \$3.3 Arithmetic Operations in x86-64
- Basic Operations
  - `addq Src, Dest`: Adds Src to Dest.
  - `subq Src, Dest`: Subtracts Src from Dest.
  - `imulq Src, Dest`: Multiplies Src and Dest.
  - `salq Src, Dest`: Shifts Dest left by Src bits (Shift arithmetic left).
  - `sarq Src, Dest`: Shifts Dest right by Src bits, maintaining sign (Shift arithmetic right).
  - `shrq Src, Dest`: Shifts Dest right by Src bits, without maintaining sign (Shift logical right).
- Special Arithmetic Instructions
  - `incq Dest`: Increments Dest by 1.
  - `decq Dest`: Decrements Dest by 1.
  - `negq Dest`: Negates the value of Dest.
  - `notq Dest`: Performs bitwise NOT on Dest.

### \$3.4 Logical Operations
- Basic Logical Instructions
  - `andq Src, Dest`: Bitwise AND of Src and Dest.
  - `orq Src, Dest`: Bitwise OR of Src and Dest.
  - `xorq Src, Dest`: Bitwise XOR of Src and Dest.
- Setting Condition Codes
  - Operations like `addq`, `subq`, and `andq` set condition codes based on the result which can be used for subsequent conditional operations.

## Address Computation Instruction
- `leaq Src, Dest`: Loads effective address from Src to Dest. Useful for address computations and pointer manipulations.

### \$3.5 Example: Arithmetic Expression Evaluation
- C Code Example:
```c
long arith(long x, long y, long z) {
    long t1 = x + y;
    long t2 = z + t1;
    long t3 = x + 4;
    long t4 = y * 48;
    long t5 = t3 + t4;
    long rval = t2 * t5;
    return rval;
}
```
- this is compiled to:
```asm
arith:
    leaq (%rdi,%rsi), %rax  # t1 = x + y
    addq %rdx, %rax         # t2 = z + t1
    leaq 4(%rdi), %rdx      # t3 = x + 4
    salq $4, %rsi           # t4 = y * 16 (left shift by 4 is equivalent to multiplying by 16)
    leaq (%rdi,%rsi), %rcx  # t5 = t3 + t4
    imulq %rcx, %rax        # rval = t2 * t5
    ret                     # return rval
```