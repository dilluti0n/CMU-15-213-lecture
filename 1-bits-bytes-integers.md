# 1. bits, bytes, and integers
> lecture source : [02-03-bits-ints.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/02-03-bits-ints.pdf)\
> lab assignment solution for Datalab : [Repository](https://github.com/dilluti0n/CMU-15213-Lab/tree/master/1_DataLab)

### \$1.1 Words
- Indicating the nominal size of integer and pointer data.
- For a `w-bit` word size, the virtual adress can range from $0$ to $2^w - 1$ for each byte.
- `32-bit` computer, which have a `32-bit` word size, the limit of virtual adress is `4 GB`(= $2^32$ bytes).
- For `64-bit` computer, which is normal at these days, it is `16 EB`(= $2^64$ bytes) in theory. But in reality, the common architecture like `x86-64` do not use this words for all memory location. it uses only 48 bits for the memory allocation, which makes it to `256 TB`.

### \$1.2 Data Sizes
- Computers and compilers support multiple data formats using different ways to encode data.
- Here Is normal Data types and sizes used in C language.

<div align = "center">

| C declaration |32-bit|64-bit|
|:--------------|:----:|:----:|
|char           |1     |1     |
|short int      |2     |2     |
|int            |4     |4     |
|long int       |4     |8     |
|long long int  |8     |8     |
|char *         |4     |8     |
|float          |4     |4     |
|double         |8     |8     |

</div>

- `char *` is a `pointer`, the data type that has the values we discussed above.
- All pointers(`int *`, `long *`, ...) are essentially the same size and have the value of one of the virtual memory addresses, but different types of pointers have different pointer arithmetic.(e.g. for type `int *` variable, `++` will increase the value of variable `4`, which is `1` for `char *`)

### \$1.3 Addressing and Byte Ordering
- multiple-byte object adress ordering rule: Big endian and Little endian type.

for example, when machine ordering 8-bit int like `0x01234567` : 
- Big endian (e.g. `Sun`)
|0x100|0x101|0x102|0x103|
|:---:|:---:|:---:|:---:|
|01|23|45|67|
- Little endian (e.g. `IA32`, `x86-64`)
|0x100|0x101|0x102|0x103|
|:---:|:---:|:---:|:---:|
|67|45|23|01|
- Example:
```c
#include <stdio.h>
typedef unsigned char *pointer;

void show_bytes(pointer start, size_t len)
{
    size_t i;
    for (i = 0; i < len; i++)
        printf("%p\t%.2x\n", start + i, start[i]);
}

main()
{
    int a = 15213;
    printf("int a = 15213;\n");
    show_bytes((pointer) &a, sizeof(int));
}
```
the result in `macOS x86-64`(little endian machine) is
```
int a = 15213;
0x7ff7b096965c	6d
0x7ff7b096965d	3b
0x7ff7b096965e	00
0x7ff7b096965f	00
```
this indicates `&a` has a value `0x7ff7bec5665c`, and the result of refrencing `printf("%i\n", (char *) &a);` is `110(0x6d)`, not `0` or `15213`.

### \$1.4 Unsigned & Signed values

- Unsigned :
  - $B2U(x_i) = \sum_{i = 0}^{w-1} 2^{i-1}x_i$
  - $0 (Umin) \leq (Unsigned) \leq 2^{w} - 1 (UMax)$
- Two's Complement Values :
  - $B2S(x_i) = -2^{w-1}x_{w-1} + \sum_{i = 0}^{w-2} 2^{i-1}x_i$
  - the leading bit is sign bit.
  - $-2^{w-1}(Tmin) \leq (Signed) \leq 2^{w-1} (TMax)
- Conversion :
  - conversion between two's complement values and unsigned values always reserve its bit pattern.
- Casting :
  - In C, If there is a mix of unsigned and signed in single expression, signed values implicitly cast to unsigned.

### \$1.5 bit-level operation in C

- I will not specifically explain the workings of each bit-level operation in C here. For more information about each operation and implication, see my lab assignment [repository](https://github.com/dilluti0n/CMU-15213-Lab/tree/master/1_DataLab).
- However, the Shift operation in C can be particularly confusing in its behavior:
  - Logical Right Shift : Fill with 0's on left
  - Arithmetic Right Shift : Replicate most significant bit on left
  - right-shifting an `unsigned` value is defined by the C standard as a `logical shift`, but for a `signed` value, it is not defined; most compilers typically perform an `arithmetic shift` implicitly.
  - Shifting amount `< 0` or `>=` word size is not defined.(In the latter case, compilers typically perform the operation as mod (size) implicitly.)
