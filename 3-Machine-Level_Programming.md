# 3. Machine Level Programming 1: Basics
> lecture source : [05-machine-basics.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/05-machine-basics.pdf)\

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