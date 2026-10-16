The **GNU Compiler Collection (GCC)** is a powerful, open-source compiler system developed by the **GNU Project**. It supports multiple programming languages, including **C, C++, Objective-C, Fortran, Ada, and more**.

### **Key Features of GCC:**

1. **Multi-language support** → Primarily used for C and C++, but supports others.
2. **Cross-platform** → Runs on Linux, macOS, and Windows (via MinGW or Cygwin).
3. **Optimizations** → Produces efficient machine code with various optimization levels.
4. **Standard compliance** → Supports ANSI C, C11, C17, and newer standards.
5. **Debugging support** → Works with `gdb` for debugging.
6. **Open-source & free** → Available under the **GNU General Public License (GPL)**.

### **Common GCC Commands for C:**

|Command|Description|
|---|---|
|`gcc file.c`|Compiles `file.c` and creates `a.out` (default output).|
|`gcc -o output file.c`|Compiles `file.c` into an executable named `output`.|
|`gcc -Wall file.c`|Enables warnings to catch errors.|
|`gcc -g file.c`|Compiles with debugging info (for `gdb`).|
|`gcc -O2 file.c`|Enables optimization for better performance.|

### **Example Usage:**

```sh
gcc -o myprogram myprogram.c
./myprogram
```

This compiles `myprogram.c` into an executable `myprogram` and then runs it.

Do you need help installing GCC or troubleshooting something? 🚀