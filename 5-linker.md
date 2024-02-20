# 5. linker
> lecture source: [13-linking.pdf](https://www.cs.cmu.edu/afs/cs/academic/class/15213-f15/www/lectures/13-linking.pdf)

### **Compile**

`*.c` --`cpp`--> `*.i` --`cc1`--> `*.s` --`as`--> `*.o` --`ld`--> `a.out`
- Compiler drivers
  - `cpp` - c preprocessor - process the `#include`, `#define`, ...
  - `cc1` - c compiler - translates `*.i` to `*.s`, assembly
  - `as` - assembler - translates assembly to relocatable obj file, `.o`
  - `ld` - linker - merges several relocatable obj files to executable obj file

### **object files**
- Relocatable object file `.o`
  - contains code and data in a form that can be combined with other relocatable object files.
- Executable object file `a.out`
  - contains code and data that can be copied directly into memory and executed.
- Shared object file `.so`
  - can be loaded into memory and linked dynamically, load time or run-time
  - `.DLL` on windows

## $5.1. Executable and Linkable Format (ELF)
one unified format for object files.

- ELF header - Word size, byte ordering, file type (`.o`, `exec`, `.so`), machine type ...
- .text - Code
- .rodata - Read only data: jump tables, string litarals, ...
- .data - initalized global C variables
- .bss 
  - Uninitialized global C variables (Block Started by Symbol)
  - occupies no actual space in obj files
  - just place holder (Better Save Space)
- .symtab
  - Symbol table
  - Procedure and static variable names, Section names and locations
- .rel.text
  - list of locations in the `.text` that will need to be modified by linker
  - instructions that call external function, reference a global variable
  - no need in `executable`
- .rel.data 
  - relocation info for `.data`
  - any initialized global variables with value of global variable's adress or externally defined function's adress need relocation.
- .debug - info for symbolic debuging (`gcc -g`)
- Section header table - offsets and sizes of each section

### **Symbols**

Each relocatable object module `m`, has a symbol table that contains info about the symbols that are defined and referenced by `m`.

In the context of a linker, there are three diffrenet kind of symbols:

- `global symbols`
  - defined by `m`, can be referenced by other modules.
  - functions and global variables without C `static` attribute.
- `externals`
  - Global symbols referenced by `m`, defined by other module.
- `local symbols`
  - defined and referenced exclusively by `m`
  - `static` fucntions and variables. (local `static` variable also stored in `.bss`, or `.data` unlikely local non-`static` variable, which is stored on the stack
  
### **Symbol table**

Symbol tables are built by assemblers, using symbols exported by compiler into the assembly-language `.s` file. An ELF symbol table contained in `.symtab` section is array of entries.

```c
typedef struct {
	int name;        /* String table offset */
	int value;       /* Section offset, or VM adress */
	int size;        /* Object size in bytes */
	char type:4;     /* Data, func, section, or src file name */
	     binding:4;  /* Local or global */
	char reserved;   /* Section header index, ABS, UNDEF, */
                     /* or COMMON */
	char section;
} Elf_Symbol;
```
  
## $5.2. linker
linker's two main tasks:
- `symbol resolution` - associate each `symbol` reference of object files with exactally one definition.
- `Relocation`
  - Relocatable obj file: code and data sections are start at adress 0
  - linker `relocates` these sections by associating a memory location with each symbol definition and then modifying all of the references to those symbols.

### **Step 1: Symbol Resolution**
- linker resolves each referance to `global symbols` in module `m` to their definitions. (whether internal or external)
- Duplicated Symbols?
  - Strong - procedures and initialized globals (`.data` section)
  - weak - uninitialized globals
  - Rules:
	1. Multiple strong symbols are not allowed (if not, linker error)
	2. Given a strong symbol and multiple weak symbols, the stromg symbol is chosen.
	3. If there are multiple weak symbols, an `arbitary` one picked; can override this with `gcc -fno-common`
		
### **Step2: Relocation**
- Mergh each module and relocate the adresses of each sections to actual `VM` adress.
- Replace the specified adresses in the `.text` and `.data` sections as indicated by `.rel.text` and `.rel.data` with the addresses of the resolved symbol definition.

## $5.3. Libraries
Basic concept: package functions commonly used by programmers.\
how?
1. Put all fuctions into a single source file
   - Link Big object file
   - Space and time *inefficient*
2. Put each function in an separate source file
   - Explicitly link appropriate binaries
   - More efficient but *burdensome* on programmer
   
### **Old-fashiond Solution: Static Libraries**
- Static libraries (`.a`, archive files)
  - Concatenate related `.o` files into a single file with an index.
  - Enhance linker so that in tries to resolve unresolved external references by looking for symboles in one or more archives.
  - If archive member file resolves reference, link it into the executable.
  - made by `ar rs libc.a atoi.o prontf.o ... random.o`
- `libc.a` and `libm.a`
  - `libc.a` - standard C library
	- 4.6 MB archive of 1496 object files.
	- I/O, memory allocation, signal handling, data and time, random, integer math
  - `libm.a` - the C math library
	- 2MB archive of 444 object files.
	- floating point math (sin, cos, tan, log, exp, sqrt, ...)
- linker's algorithm for resolving archive files
  1. scan `.o` files and `.a` files in the command line order
  1. During the scan, keep a list of current unresolved refernces
  1. As each new obj file encountered, try to resolve each unresolved reference in the list against the symbols defined in obj files.
  1. if any entries in the unresolved list at end of scan, error.

**problems of Static Libraries**

- unresolved refernces only produced by `.o` files; if command line order of `.a` file is before the referncing `.o` file, it occurs error every time.

```bash
$ gcc -L. libtest.o -lmine
$ gcc -L. -lmine libtest.o
libtest.o: In function `main':
libtest.o(.test+0x4): undefined reference to `libfun'
```

> `-L.`: Instructs the compiler to look in `./` for the library file.\
> `-limne`: same as passing argument `libmine.a`

if `-lmine` (or `libmine.a`) introduced first to the linker, it has no idea what refernces are unresolved in `libtest.o`.

- Duplication in the stored\running executables (size issue)
- Minor bug fixes of system libraries require each application to explicitly relink.

### **Mordern Solution: Shared Libraries**
- loaded and linked into an application *dynamically*, at either load-time or run-time
- `.dll` (window, dinamic link library), `.so` (linux, shared objects)
- to make: `gcc -shared -o libvecter.so addvec.c multvec.c`

**load-time linking**
- On Linux, it handled automatically by the dynamic linker `ld-linux.so`
- Standard C library `libc.so` usually dynamically linked.
1. Linker `ld` puts relocation and symbol table info in shared libreary onto relocatable object files.

```bash
$ gcc -o libtest libtest.c ./libvector.so
```

2. When executable object file executed (by module `execve`), Dynamic linker `ld-linux.so` Fully linkes code and data (on `libc.so` and `./libvector.so`) and loads Fully linked executable to the memory.

```bash
$ ./libtest
```

**Run-time linking**
> use `defcn.h`

```c
#include <stdio.h>
#include <stdilb.h>
#include <dlfcn.h>

int x[2] = {1, 2};
int y[2] = {3, 4};
int z[2];

int main()
{
  void *handle;
  void (*addvec)(int *, int *, int *, int);
  char *error;

  /* Dynamically load the shared library that contains addvec() */
  handle = dlopen("./libvector.so", RTLD_LAZY);
  if (!handle) {
    fprintf(stderr, "%s\n", dlerror());
    exit(1);
  }
  /* ... */
  /* get a pointer to the addvec() we just loaded */
  addvec = dlsym(handle, "addvec");
  if ((error = dlerror()) != NULL) {
    fprintf(stderr, "%s\n", error);
    exit(1);
  }
  /**
   * call the dynamic linked function addvec()
   * defined on ./libvector.so
   */
  addvec(x, y, z, 2);
  printf("z = [%d %d]\n", z[0], z[1]);

  /* Unload the shared libreary */
  handle = dlopen("./libvector.so", RTLD_LAZY);
  if (!handla) {
    fprintf(stderr, "%s\n", error);
    exit(1);
  }
}
```

Summery
-
- Linking is technique that allows programs to be constructed from multiple object files.
- Linking can happen at different times in an program's lifetime:
  - compile time
  - Load time
  - Run time