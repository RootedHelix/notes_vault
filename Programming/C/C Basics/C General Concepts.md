Related to [[C Language]]
#c_general_concepts

- In C, **any non-null pointer evaluates to `true`**, and a **NULL pointer evaluates to `false`**.
## Data Type memory Sizes
#datatype_size_c 

Got you. Here’s a compact, “what size is what” map—grouped by **primitive kinds**, then shown for the three common data models you’ll meet: **ILP32 (32-bit)**, **LP64 (Unix-like 64-bit)**, **LLP64 (Windows 64-bit)**.

### Core primitive types
Here you go—added a **Value Range** column (sizes in **bytes**; integer ranges assume two’s-complement, `CHAR_BIT == 8`).

| Type                               |     ILP32 (32-bit) |                    LP64 (Linux/macOS 64-bit) | LLP64 (Windows 64-bit) | Value Range (typical)                                                                                                                                                                         |
| ---------------------------------- | -----------------: | -------------------------------------------: | ---------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `void`                             |                  — |                                            — |                      — | N/A                                                                                                                                                                                           |
| `_Bool`                            |                  1 |                                            1 |                      1 | {0, 1}                                                                                                                                                                                        |
| `char`*                            |                  1 |                                            1 |                      1 | **implementation-defined**: −128…127 **or** 0…255                                                                                                                                             |
| `signed char` / `unsigned char`    |                  1 |                                            1 |                      1 | −128…127 **/** 0…255                                                                                                                                                                          |
| `short` / `unsigned short`         |                  2 |                                            2 |                      2 | −32,768…32,767 **/** 0…65,535                                                                                                                                                                 |
| `int` / `unsigned int`             |                  4 |                                            4 |                      4 | −2,147,483,648…2,147,483,647 **/** 0…4,294,967,295                                                                                                                                            |
| `long` / `unsigned long`           |                  4 |                                            8 |                      4 | ILP32: −2,147,483,648…2,147,483,647 **/** 0…4,294,967,295; LP64: −9,223,372,036,854,775,808…9,223,372,036,854,775,807 **/** 0…18,446,744,073,709,551,615; LLP64: same as ILP32                |
| `long long` / `unsigned long long` |                  8 |                                            8 |                      8 | −9,223,372,036,854,775,808…9,223,372,036,854,775,807 **/** 0…18,446,744,073,709,551,615                                                                                                       |
| `float`                            |                  4 |                                            4 |                      4 | (IEEE-754 binary32) ~1.17549435e−38…3.40282347e38 (≈7 decimal digits)                                                                                                                         |
| `double`                           |                  8 |                                            8 |                      8 | (IEEE-754 binary64) ~2.225073858e−308…1.797693134e308 (≈15–17 digits)                                                                                                                         |
| `long double`**                    | 12/16 (target-dep) | usually 16 (SysV); sometimes 8/16 target-dep |                      8 | **Varies by target/ABI**: 80-bit extended (stored in 16 B) ~3.36e−4932…1.19e4932 (≈18–19 digits); binary128 (quad) ~6.48e−4966…1.19e4932 (≈33–36 digits); or just binary64 (same as `double`) |

* Plain `char`’s signedness is implementation-defined.  
**`long double` notes:**

- **LP64 (x86-64 SysV/Linux):** commonly 80-bit extended precision with 16-byte storage.
    
- **LP64 (aarch64 glibc):** often IEEE-754 binary128.
    
- **Windows LLP64 (MSVC):** `long double` == `double` (binary64).
* `char` signedness is implementation-defined (signed on some, unsigned on others).  
** On x86-64 SysV (most Linux), `long double` is 16 bytes (80-bit precision stored in 16). On Windows it’s 8 bytes.

### Pointers / references

| Type                                    | ILP32 | LP64 | LLP64 |
| --------------------------------------- | ----: | ---: | ----: |
| Object/function pointer (e.g., `void*`) |     4 |    8 |     8 |
| `size_t` / `ptrdiff_t`                  |     4 |    8 |     8 |
| `intptr_t` / `uintptr_t`                |     4 |    8 |     8 |

### Character & text related

| Type             |                              ILP32 |          LP64 |       LLP64 |
| ---------------- | ---------------------------------: | ------------: | ----------: |
| `wchar_t`        | 4 (GCC/Clang on Linux) or 2 (some) | 4 (Unix-like) | 2 (Windows) |
| `char16_t` (C11) |                                  2 |             2 |           2 |
| `char32_t` (C11) |                                  4 |             4 |           4 |

### Enumerations

| Type   |                     ILP32 |      LP64 |     LLP64 |
| ------ | ------------------------: | --------: | --------: |
| `enum` | usually 4 (same as `int`) | usually 4 | usually 4 |

### Fixed-width integers (`<stdint.h>`)

These are **guaranteed sizes** regardless of 32/64-bit:

|Type|Size|
|---|--:|
|`int8_t` / `uint8_t`|1|
|`int16_t` / `uint16_t`|2|
|`int32_t` / `uint32_t`|4|
|`int64_t` / `uint64_t`|8|
|`intmax_t` / `uintmax_t`|≥ largest native int (commonly 8)|

### OS/ABI-defined “biggies” you’ll meet (POSIX)

|Type|ILP32|LP64|LLP64|
|---|--:|--:|--:|
|`off_t` (file offsets)|4 by default; **8 with** `-D_FILE_OFFSET_BITS=64`|8|8|
|`ssize_t`|4|8|8|
|`time_t`|often 4 (unless opted into 64-bit time)|8 (modern)|8|

---

### Practical tips (especially for your C systems/server work)

- For **on-disk / wire formats**, use **fixed-width** types (`uint32_t`, `uint64_t`) and explicit endianness.
    
- Keep **pointers and `size_t`** for in-process memory/indices only; never serialize pointers.
    
- Be mindful that `long` differs: **LP64 → 8 bytes**, **LLP64 → 4 bytes**.
    
- Prefer printing with the right specifiers: `size_t`→`%zu`, `ptrdiff_t`→`%td`, `uint64_t`→`PRIu64` from `<inttypes.h>`.
    
- Expect **larger memory footprint** on 64-bit (pointers, page tables), but more registers and better ISA usually mean **better performance** for heavy workloads.

## Format Specifiers
Here’s a compact, “actually useful” list. I split it into **`printf`-style output**, **`scanf`-style input**, and **length modifiers** (to combine with specifiers).

### `printf` / `fprintf` / `snprintf`

|Specifier|Meaning / Typical Type Printed|
|---|---|
|`%d`, `%i`|signed int (base-10)|
|`%u`|unsigned int (base-10)|
|`%o`|unsigned int (octal)|
|`%x`, `%X`|unsigned int (hex)|
|`%f`, `%F`|double (fixed-point)|
|`%e`, `%E`|double (scientific)|
|`%g`, `%G`|double (shorter of `%f`/`%e`)|
|`%a`, `%A`|double (hexadecimal FP)|
|`%c`|int as an unsigned char (a character)|
|`%s`|`char*` C-string (NUL-terminated)|
|`%p`|pointer (`void*` recommended to cast)|
|`%n`|write the number of characters printed so far into an `int*` (dangerous if misused)|
|`%%`|a literal `%`|

**Common length-modified forms in `printf`:**

- `%ld`, `%lld` → `long`, `long long`
    
- `%zd` / `%zu` → `size_t` (from C99)
    
- `%td` / `%ti` → `ptrdiff_t`
    
- `%jd` / `%ju` → `intmax_t`/`uintmax_t` (from `<inttypes.h>`)
    
- Use `<inttypes.h>` macros for fixed-width integers: e.g., `PRId32`, `PRIu64`, `PRIuPTR`.
    

### `scanf` / `fscanf` / `sscanf`

(_These read **into pointers**; the letter means the **destination type**._)

|Specifier|Stores into…|
|---|---|
|`%d`, `%i`|`int*` (base-10 for `%d`; `%i` auto-base: 0x hex, 0 octal)|
|`%u`|`unsigned int*`|
|`%o`|`unsigned int*` (octal)|
|`%x`, `%X`|`unsigned int*` (hex)|
|`%f`|`float*`|
|`%lf`|`double*` (**scanf only**; in `printf`, `%lf` ≡ `%f`)|
|`%Lf`|`long double*`|
|`%e`, `%E`, `%g`, `%G`, `%a`, `%A`|same rules as `%f`/`%lf`/`%Lf` for float/double/long double pointers|
|`%c`|`char*` (reads exactly one char; use width like `%3c`)|
|`%s`|`char*` (reads a word; stops at whitespace)|
|`%[...]`|scanset; e.g. `%[^\n]` reads until newline|
|`%p`|`void**` (reads a pointer value)|
|`%n`|`int*` (number of chars read so far)|
|`%%`|matches a literal `%`|

**Safety tips for `scanf`:**

- Always bound strings: `scanf("%99s", buf);` for `char buf[100];`
    
- For lines with spaces use scanset: `scanf(" %199[^\n]", buf);`
    
- Prefer `fgets` + `sscanf` for complex input.
    

### Length modifiers (combine with specifiers)

|Modifier|Use with|Meaning (input or output)|
|---|---|---|
|`hh`|`d/i/u/o/x`|`signed/unsigned char` (`scanf`: `char*`; `printf`: promoted from `int`)|
|`h`|`d/i/u/o/x`|`short`/`unsigned short`|
|_(none)_|—|plain `int`/`unsigned`|
|`l`|`d/i/u/o/x`|`long`/`unsigned long`|
|`ll`|`d/i/u/o/x`|`long long`/`unsigned long long`|
|`j`|`d/i/u/o/x`|`intmax_t`/`uintmax_t`|
|`z`|`d/i/u/o/x`|`size_t`/`ssize_t` (`%zu`, `%zd`)|
|`t`|`d/i/u/o/x`|`ptrdiff_t` (`%td`)|
|`L`|`f/e/g/a`|`long double` (`%Lf`)|
|`l` with `%c`/`%s`|wide char/string: `%lc` (`wint_t`), `%ls` (`wchar_t*`)||

### Flags, width, precision (for `printf`)

- **Flags:** `- + 0 # space`
    
- **Width:** number, or `*` to take from an `int` argument.
    
- **Precision:** `.%d` or `.*`.
    
    - Examples: `%08x`, `%-10s`, `%.3f`, `%#x`, `%.*s`.
        

---

If you want, I can turn this into a printable one-pager with examples for each (and a few gotchas like `%p` needing a `(void*)` cast in `printf`).## Printing Behaviors

If you try to store more than a single character, it will only print the last character:

char myText = 'Hello';  
printf("%c", myText);

To store multiple characters (or whole words), use [strings](https://www.w3schools.com/c/c_strings.php) (which you will learn more about in a later chapter):

char myText[ ] = "Hello";  
printf("%s", myText);

- when the format specifier is set to 0 decimal places printf("%.0lf", myDouble);
		Since it will print with 0 decimal places, the print function rounds the number


### **Address-of Operator**
   - The `&` operator is also used to retrieve the **address of a variable** (i.e., a pointer to the variable).
   - Example:
     ```c
     int x = 10;
     int *ptr = &x;  // 'ptr' now holds the address of 'x'
     ```

### Summary Table

| Use Case             | Operator | Description                                |
| -------------------- | -------- | ------------------------------------------ |
| Bitwise AND          | `&`      | Operates on each bit of two operands       |
| Address-of (Pointer) | `&`      | Retrieves the memory address of a variable |
|                      |          |                                            |
|                      |          |                                            |

Let me know if you'd like a deeper dive into pointers or bitwise operations!


## Struct Pointer Dereference Operator ( -> )

The `->` operator is called the **struct pointer dereference operator** in C. It is used to **access members of a struct** through a **pointer** to that struct.

When you have a pointer to a struct, you can't use the dot (`.`) operator directly to access its members. Instead, you use the `->` operator to both dereference the pointer and access the struct's members.

why would I want to access a struct with a pointer rather than accessing the struct directly

###### ChatGPT said:

Accessing a struct using a pointer instead of accessing it directly has several advantages:

---

### **1. Efficient Memory Usage**

- When you pass a struct directly to a function, a **copy of the entire struct** is made.
- If the struct is large, copying it can be inefficient.
- Using a pointer allows the function to access the original struct **without copying it**.
#include <stdio.h>

typedef struct {
    int id;
    char name[50];
} Student;

// Pass by value (makes a copy)
void printStudentValue(Student s) {
    printf("Student ID: %d, Name: %s\n", s.id, s.name);
}

// Pass by pointer (no copy)
void printStudentPointer(Student *s) {
    printf("Student ID: %d, Name: %s\n", s->id, s->name);
}

int main() {
    Student s1 = {1, "Alice"};

    printStudentValue(s1);  // Passes a copy
    printStudentPointer(&s1);  // Passes a pointer (efficient)

    return 0;
}

## Type Conversion

Two types of Type conversion in C: 
*Implicit* conversion (Automatically)
*Explicit* Conversion (Manually)

Implicit Conversion Example: 
// Automatic conversion: int to float  
float myFloat = 9;  
  
printf("%f", myFloat); // 9.000000

// Automatic conversion: float to int  
int myInt = 9.99;  
  
printf("%d", myInt); // 9

Explicit Conversion Example: 
int num1 = 5;  
int num2 = 2;  
float sum = (float) num1 / num2;  
  
printf("%.1f", sum); // 2.5


## Variable Values

if you assign a new value to an existing variable, it will overwrite the previous value

int myNum = 15; //myNum is 15
myNum = 10; // Now myNum is 10

## Determining whether or not to use void or int in a function

The purpose of return types in C (and other programming languages) is to define the type of value a function will produce and send back to the calling code. This helps the compiler enforce correct usage and provides a way for functions to communicate results or outcomes to other parts of a program.

Here’s why return types are important:

---

### **1. Communicating Results**
Return types allow a function to send data back to the caller for further use or decision-making.

- Example: A function returning the sum of two integers.
  ```c
  int add(int a, int b) {
      return a + b;
  }

  int main() {
      int result = add(5, 10);  // result gets the returned value
      printf("Sum: %d\n", result);
      return 0;
  }
  ```

---

### **2. Enforcing Type Safety**
The return type ensures that the caller receives the correct type of data and helps avoid unintended type errors.

- Example: If a function returns a `float`, the caller can expect a floating-point number, and the compiler will warn if the types mismatch.
  ```c
  float divide(int a, int b) {
      return (float)a / b;
  }

  int main() {
      float result = divide(10, 3);  // Correct type match
      printf("Result: %.2f\n", result);
      return 0;
  }
  ```

---

### **3. Enabling Reusability**
Functions with return types are versatile and reusable because they can compute and return results for different inputs without hardcoding specific operations or values.

- Example: A reusable function that calculates the square of a number.
  ```c
  int square(int x) {
      return x * x;
  }
  ```

---

### **4. Error Signaling**
Return types, especially integers and enums, are often used to signal success, failure, or specific error codes.

- Example: A function signaling success or failure.
  ```c
  int performTask() {
      if (/* some condition */) {
          return 0;  // Success
      }
      return -1;  // Error
  }

  int main() {
      int status = performTask();
      if (status == 0) {
          printf("Task succeeded.\n");
      } else {
          printf("Task failed.\n");
      }
      return 0;
  }
  ```

---

### **5. Supporting Complex Data Exchange**
Functions can return complex data structures like `structs` or pointers, allowing for the transfer of multiple values or large datasets.

- Example: Returning a structure to convey multiple results.
  ```c
  struct Point {
      int x, y;
  };

  struct Point createPoint(int x, int y) {
      struct Point p = {x, y};
      return p;
  }
  ```

---

### **6. Enhancing Clarity and Maintainability**
By explicitly specifying a return type, the purpose and expected outcome of a function become clear, improving code readability and maintainability.

- Example: A function named `getStatus()` returning an enum type makes it clear what to expect.
  ```c
  enum Status { SUCCESS, FAILURE };

  enum Status getStatus() {
      return SUCCESS;
  }
  ```

---

In summary, **return types are critical for defining what a function does, ensuring correctness, improving code quality, and enabling robust communication between functions**.

## Return Types
 [ Void ]

### **General Guidelines**

- **Void Functions**: Use `void` when a function exists solely for side effects (e.g., printing, modifying global or pointer variables, or setting up states).
- **Value-Returning Functions**: Use a return type when the result or feedback from the function is needed by the caller.
- **Error-Sensitive Operations**: Consider returning a status code or result even for non-computational functions, especially when failures are possible.

- Variable names are known as **Identifiers**
- constant -  which means **unchangeable** and **read-only**:

## Bitwise Operators
#bitwise_operators_c


Bitwise operators are powerful tools for low-level programming. They provide direct control over binary data and are commonly used for:

- Masking bits
- Setting or clearing specific bits
- Bit-level arithmetic
- Optimizing performance-critical code

### Bitwise Not Operator

- the ( ~ ) operator flips the bits in of a number

- most modern CPU's 0operate the fastest when they read/write with 8 bytes chunks
- Misaligned memory can cause:
- Slower memory access (because CPU must do extra work)
    
- Crashes on some strict architectures (e.g., ARM, SPARC)
### Bitwise & operator
[[Bitwise & Operator]]
- The ( & ) operate compares two bit sets and sets the bits to 1 if and only if they are both 1, otherwise they are set to 0

- The `&` operator performs a **bitwise AND** operation on two integers. Each bit of the result is 1 if the corresponding bits of both operands are also 1; otherwise, it is 0.
   - Example:
     ```c
     int a = 5;  // Binary: 0101
     int b = 3;  // Binary: 0011
     int c = a & b;  // Binary: 0001 (result is 1)
     ```




### Bitwise OR
- `|` (OR)   Compares two bit sets and returns a new integer where each bit is 1 if either of the corresponding bits in the opperand is 1. 
[[Bitwise OR]]

### Bitwise XOR 
[[Bitwise XOR]]
- `^` (XOR) operator compares two integers bit by bit and returns a new integer where each bit is: 1 if the bits are different and 0 if the bits are the same
    

### Bitwise Left Shift
- `<<` (left shift) Bitwise left shift shifts all the bits in a number to the left by a sepcified number of positions, filling in zeros on the right.
- for a number x << n,  x * (2ⁿ)


### Bitwise Right Shift

- `>>` (right shift) Bitwise right shift,
- x >> n equates to roughly x / (2ⁿ)

- x             = 00001100   (12)
  x >> 2    = 00000011     (3)
### BitFields
- A way to store moultiple boolean or small-range values compactly in a single variable by assigning each value to specific bits. Used to save memory or control hardware registers efficiently

## How to Run Multiple .c Executable Files

To use multiple `.c` files in a project within **CLion IDE**, you need to structure your project and configure the `CMakeLists.txt` file properly. CLion relies on **CMake** as its build system, and you can specify which `.c` files to compile.

Here’s a step-by-step guide:

---

### 1. **Create a New Project**
- Open CLion and create a new project:
  - **File > New Project > C Executable**.
  - Select your compiler and specify a location for the project.

---

### 2. **Add Multiple `.c` Files**
- In the `src` directory (or project root), create or add your `.c` files. For example:
  ```
  project_root/
  ├── CMakeLists.txt
  ├── src/
      ├── main.c
      ├── file1.c
      └── file2.c
  ```
- Each `.c` file will contain different parts of your program, such as functions or logic.

---

### 3. **Edit the `CMakeLists.txt` File**
You must specify all source files in the `add_executable()` command within `CMakeLists.txt`.

#### Example `CMakeLists.txt`:
```cmake
cmake_minimum_required(VERSION 3.20)
project(MyProject C)

set(CMAKE_C_STANDARD 99)

# Add all .c files here
add_executable(MyProject src/main.c src/file1.c src/file2.c)
```

This tells CMake to compile all the specified `.c` files and link them together.

---

### 4. **Define Functions in the Additional Files**
Ensure each `.c` file has functions that can be used in `main.c`. For example:

#### `file1.c`:
```c
#include <stdio.h>

void greet() {
    printf("Hello from file1.c!\n");
}
```

#### `file2.c`:
```c
#include <stdio.h>

void farewell() {
    printf("Goodbye from file2.c!\n");
}
```

#### `main.c`:
```c
#include <stdio.h>

// Declare functions from other .c files
void greet();
void farewell();

int main() {
    greet();
    farewell();
    return 0;
}
```

---

### 5. **Build and Run the Project**
- Click **Build > Build Project** or press `Ctrl + F9`.
- Run the project by selecting **Run > Run 'MyProject'** or pressing `Shift + F10`.

---

### 6. **Optional: Use Header Files**
For better organization, create `.h` files to declare functions and include them where needed.

#### Example:

- **File Structure:**
  ```
  project_root/
  ├── CMakeLists.txt
  ├── src/
      ├── main.c
      ├── file1.c
      ├── file1.h
      ├── file2.c
      └── file2.h
  ```

- **file1.h:**
  ```c
  #ifndef FILE1_H
  #define FILE1_H

  void greet();

  #endif
  ```

- **file2.h:**
  ```c
  #ifndef FILE2_H
  #define FILE2_H

  void farewell();

  #endif
  ```

- **file1.c:**
  ```c
  #include "file1.h"
  #include <stdio.h>

  void greet() {
      printf("Hello from file1.c!\n");
  }
  ```

- **main.c:**
  ```c
  #include "file1.h"
  #include "file2.h"

  int main() {
      greet();
      farewell();
      return 0;
  }
  ```

- Update the `CMakeLists.txt` file:
  ```cmake
  add_executable(MyProject src/main.c src/file1.c src/file2.c)
  ```

---

By following these steps, CLion will compile and link multiple `.c` files into a single executable.


## Signed and Unsigned Integers

#### **What is Overflow in C?**
**Overflow** in C occurs when a variable is assigned a value that exceeds its allowed range. This causes the value to wrap around in an **unexpected** way, leading to incorrect results.

---

### **1. Signed Integer Overflow**
For **signed integers** (e.g., `int`, `long`), overflow happens when a value goes **beyond the maximum** or **below the minimum** representable number.

#### **Example: Overflow in Signed Integers**
```c
#include <stdio.h>

int main() {
    int x = 2147483647;  // Max value for a signed 32-bit int
    printf("x = %d\n", x);

    x = x + 1;  // Overflow occurs here
    printf("After overflow: x = %d\n", x); // Unexpected negative value!

    return 0;
}
```
### **Expected Output (on a 32-bit system):**
```
x = 2147483647
After overflow: x = -2147483648
```
💡 **Why?**  
- `2147483647 + 1` **wraps around** to `-2147483648` because signed integers use **two’s complement representation**.
- The highest bit (MSB) flips from `0` to `1`, making the number negative.

🔴 **Undefined Behavior:**  
According to the C standard, signed integer overflow is **undefined behavior** (UB). This means:
- The result **is not guaranteed** to be predictable.
- Different compilers might handle it differently.
- Some optimizations may assume overflow **never happens**, leading to unexpected issues.

---

### **2. Unsigned Integer Overflow**
For **unsigned integers** (e.g., `unsigned int`, `unsigned long`), overflow behaves **differently** because there is no sign bit. Instead, numbers wrap around from the maximum value back to `0`.

#### **Example: Overflow in Unsigned Integers**
```c
#include <stdio.h>

int main() {
    unsigned int x = 4294967295;  // Max value for a 32-bit unsigned int
    printf("x = %u\n", x);

    x = x + 1;  // Overflow occurs here
    printf("After overflow: x = %u\n", x); // Wraps back to 0

    return 0;
}
```
### **Expected Output (on a 32-bit system):**
```
x = 4294967295
After overflow: x = 0
```
💡 **Why?**  
- `4294967295 + 1` **wraps around** to `0` because there is **no sign bit**.
- This behavior is **well-defined** in C—unsigned integers always wrap around modulo `2^n`.

---

### **3. Overflow in Arithmetic Operations**
### **Multiplication Overflow**
If you multiply two large numbers, the result may exceed the storage limit.

```c
#include <stdio.h>

int main() {
    int a = 100000, b = 100000;
    int result = a * b;  // Overflow occurs here
    printf("Result: %d\n", result);  // Unexpected value!
    return 0;
}
```
### **Expected Issue:**
- Instead of `10,000,000,000`, the result **wraps around**, giving a garbage value.

💡 **Solution:** Use `long long` or `unsigned long long` for larger values.

---

### **4. Detecting and Preventing Overflow**
### **Using Conditional Checks**
To prevent overflow, check if an operation exceeds the type's maximum value **before performing it**.

```c
#include <stdio.h>
#include <limits.h>

int main() {
    int a = INT_MAX, b = 10;
    
    if (a > INT_MAX - b) {  // Prevent overflow
        printf("Overflow detected!\n");
    } else {
        int result = a + b;
        printf("Safe result: %d\n", result);
    }

    return 0;
}
```

### **Using Larger Data Types**
For large numbers, use `long long` or `unsigned long long`.

```c
#include <stdio.h>

int main() {
    long long a = 1000000000LL;
    long long b = 1000000000LL;
    long long result = a * b;
    
    printf("Result: %lld\n", result);  // Safe multiplication
    return 0;
}
```

---

### **5. Summary**
| Type | Overflow Behavior |
|------|------------------|
| **Signed Integer (`int`)** | Undefined behavior (wraps in two’s complement) |
| **Unsigned Integer (`unsigned int`)** | Well-defined wrap-around (modulo `2^n`) |
| **Floating Point (`float`, `double`)** | Becomes `∞` (Infinity) instead of wrapping |

### **Key Takeaways**
✅ **Signed integer overflow is undefined behavior.**  
✅ **Unsigned integers wrap around predictably.**  
✅ **Use checks (`if` conditions) or larger data types (`long long`) to prevent overflow.**  

Would you like examples of overflow handling in real-world applications? 🚀



## Difference between = and ==

In C, `=` and `==` are **completely different** operators:

|Operator|Meaning|Type|Usage|
|---|---|---|---|
|`=`|**Assignment operator**|Used to assign values to variables|`a = 10;`|
|`==`|**Equality comparison operator**|Used to compare two values|`if (a == 10) { ... }`|


## Macros in C

- use #define

(ie.)
#define type_of(x) _Generic((x), \  
int: "int", /
float: "float", /
double: "double", \  
int*: "int pointer", \  
default: "unknown")

## Memory Address

- memory address assigned to variable at declaration
- Memory when printed is in **==hexidecimal==** form
## Pointers

	Line:548 
- A pointer is a variable that stores the memory address of another variable as its value
- points to a data type of the same type
- created with the * operator
- two ways to create a pointer in c:
		int* myNum;
		int "Asterik"myNum;

### Dereference

	Line:563
- You can use the dereference operator ' * ' to get the value that the pointer points to
- when the * operator is used outside of declaration (int* ptr) it acts as a dereference operator

### Pointers and Arrays
	Line:567
- The name of an array is a pointer to the first element of the array
		- holds the memory address of the first element in the array

### Pointer Arguments

- an expected pointer argument needs to have an address passed to it while an expected double pointer argument requires an address of a pointer that points to the desired variable

|Function Expects|You Have|What to Pass|Why|
|---|---|---|---|
|`int*`|`int`|`&x`|Needs address of `int`|
|`int*`|`int*`|`x`|Already a pointer|
|`int*`|`int*`|❌ `*x`|That’s an `int`, not a pointer|
|`int**`|`int*`|`&x`|Needs address of pointer|

### Pointer Behavior

If you are passing a double pointer in as an argument you should look to interact with that pointer inside the function with (asterik)p.

Passing pointers as arguments allows you to manipulate data outside the scope of the function. Therefore, allowing you to return multiple values.

essentially we need a new pointer within the function and with an additional level of indirection to be assigned to the callers value and then from there we can change the callers value
## Memory Management

- C has two types of memory: Static and Dynamic memory

### Static Memory

- Static memory is memory that is reserved for variables **before** the program runs. Allocation of static memory is also known as ==_compile time_== memory allocation.
- C automatically allocates memory for every variable when the program is compiled.
- For example, if you create an integer array of 20 students (e.g. for a summer semester), C will reserve space for 20 elements which is typically 80 bytes of memory (20 * 4):

### Dynamic Memory

- Dynamic memory is memory that is allocated **after** the program starts running. Allocation of dynamic memory can also be referred to as ==_runtime_== memory allocation.
- Unlike with static memory, you have full control over how much memory is being used at any time. You can write code to determine how much memory you need and allocate it.
- Dynamic memory does not belong to a variable, it can only be accessed with pointers.
- 

### Stack [ Automatic (Static Lifetime) ] vs. Heap Memory [ Dynamic (Manual Management) ] 

- **Stack**: For regular variables, which are automatically allocated and freed when they go out of scope.

	- The stack is loosely referred to as dynamic memory as the stack shrinks and grows as the program runs 
	- which is reserved for variables that are declared inside functions. Variables declared inside a function use stack memory rather than static memory.

	- When a function is called, stack memory is allocated for the variables in the function. When the function returns the stack memory is freed.

	- It is good to be aware of stack memory to be able to handle the memory usage of nested function calls and recursion. Recursion that repeats itself too many times may take up too much stack memory. When that happens it is called a **stack overflow**.

- **Heap**: For dynamically allocated memory (via `malloc()`/`calloc()`), which needs to be explicitly freed using `free()` to avoid memory leaks.

### Reallocate Memory

The `realloc()` function takes two parameters:

int *ptr2 = realloc(ptr1, size);

- The first parameter is a pointer to the memory that is being resized.
- The second parameter specifies the new size of allocated memory, measured in bytes.


- The `realloc()` function tries to resize the memory at _ptr1_ and return the same memory address. If it cannot resize the memory at the current address then it will allocate memory at a different address and return the new address instead.

**Note:** When `realloc()` returns a different memory address, the memory at the original address is no longer reserved and it is not safe to use. When the reallocation is done it is good to assign the new pointer to the previous variable so that the old pointer cannot be used accidentally.

### Deallocate Memory

When you no longer need a block of memory you should deallocate it. Deallocation is also referred to as "freeing" the memory.

Dynamic memory stays reserved until it is deallocated or until the program ends.

Once the memory is deallocated it can be used by other programs or it may even be allocated to another part of your program.

*It is considered a good practice to set a pointer to `NULL` after freeing memory so that you cannot accidentally continue using it.*

*If you continue using memory after it has been freed you may corrupt data from other programs or even another part of your own program.*

### Memory Leaks

Memory leaks occur when dynamic memory is allocated but never freed

if a memory leak happens in a loop or in a function that is called frequently it could take up too much memory and cause the computer to slow down

There is a risk of a memory leak if a pointer to dynamic memory is lost before the memory can be freed. This can happen accidentally, so it is important to be careful and keep track of pointers to dynamic memory. 

	Here are some examples of how a pointer to dynamic memory may be lost.

The pointer gets lost when reallocation fails:

int* ptr;  
ptr = malloc(sizeof(*ptr));  
ptr = realloc(ptr, 2  asterisk sizeof( asterisk ptr)); 

If `realloc()` is unable to reallocate memory it will return a pointer to `NULL` and the original memory will remain reserved.

In this example, if `realloc()` fails then the `NULL` pointer is assigned to the `ptr` variable, overwriting the original memory address so that it cannot be accessed anymore.

### Fragmentation

- In the context of memory pool allocators, fragmentation occurs when within a certain segment in memory, free memory is split by used memory making it impossible to allocate large chunks of memory because the available free memory is split and not contiguous.
-




## Function Format

For code optimization, it is recommended to separate the declaration and the definition of the function.

You will often see C programs that have function declaration above `main()`, and function definition below `main()`.

This will make the code better organized and easier to read:

## File writing, creation, and reading

- `FILE` is basically a data type, and we need to create a pointer variable to work with it (`fptr`). For now, this line is not important. It's just something you need when working with files.

To actually open a file, use the `fopen()` function, which takes two parameters:

| Parameter  | Description                                                                                                                                                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| _filename_ | The name of the actual file you want to open (or create), like `filename.txt`                                                                                                                    |
| _mode_     | A single character, which represents what you want to do with the file (read, write or append):<br><br>`w` - Writes to a file  <br>`a` - Appends new data to a file  <br>`r` - Reads from a file |

### Create File

To create a file, you can use the `w` mode inside the `fopen()` function.

The `w` mode is used to write to a file. **However**, if the file does not exist, it will create one for you


### Closing the file

Did you notice the `fclose()` function in our example above?

This will close the file when we are done with it.

It is considered as good practice, because it makes sure that:

- Changes are saved properly
- Other programs can use the file (if you want)
- Clean up unnecessary memory space

### Write to File

Let's use the `w` mode from the previous chapter again, and write something to the file we just created.

The `w` mode means that the file is opened for **writing**. To insert content to it, you can use the `fprintf()` function and add the pointer variable (`fptr` in our example) and some text:

### Example

FILE *fptr;  
  
// Open a file in writing mode  
fptr = fopen("filename.txt", "w");  
  
// Write some text to the file  
fprintf(fptr, "Some text");  
  
// Close the file  
fclose(fptr);

### Append to File

If you want to add content to a file without deleting the old content, you can use the `a` mode.

The `a` mode appends content at the end of the file:

### Example

FILE *fptr;  
  
// Open a file in append mode  
fptr = fopen("filename.txt", "a");  
  
// Append some text to the file  
fprintf(fptr, "\nHi everybody!");  
  
// Close the file  
fclose(fptr);


## Enumeration

An **enum** is a special type that represents a group of constants (unchangeable values).

Enum is short for "enumerations", which means "specifically listed".

To create an enum, use the `enum` keyword, followed by the name of the enum, and separate the enum items with a comma: 

enum Level {  
  LOW,  
  MEDIUM,  
  HIGH  
};

#### Why And When To Use Enums?

Enums are used to give names to constants, which makes the code easier to read and maintain.

Use enums when you have values that you know aren't going to change, like month days, days, colors, deck of cards, etc.

## Using User Defined Headers

If `Server.h` is a **user-defined** header, use **double quotes**:

#include "Server.h"

Angle brackets `<Server.h>` are used for **standard library headers**, which `Server.h` is not.

## Deep Copy vs Shallow Copy

- **Shallow Copy** → Copies the struct **as a whole**, including the pointer references.
    
- **Deep Copy** → Requires you to **manually copy each field**, ensuring that dynamically allocated data is duplicated into separate memory locations.

## Modifying Variables and Pointers


|**What You Modify**|**Pointer Level Needed**|**Example**|
|---|---|---|
|**Fields of a struct (`->next`)**|`T*`|`node_to_insert->next = *head;`|
|**A pointer itself (`head` of a list)**|`T**`|`*head = node_to_insert;`|
|**A pointer inside a function (`malloc`)**|`T**`|`*node_to_insert = malloc(...);`|

## Translation unit and Include Guard

#ifndef LINKEDLIST_H  
#define LINKEDLIST_H

- A translation unit is a .c file with all the the header files it includes
- the include guard  ensures that there is only one instance of the header file per translation unit, otherwise there would be errors



## Link Time Interposition (static linking)

**Interposition** is the process in C where you arrange for the linker or dynamic loader to resolve your custom function or library **before** the original one, allowing your version to be called in place of the standard implementation (library)

## Static Prefix

### 1. **File-Scope Static** (Outside Functions)

// At file level (outside any function)  
static lpd_open_file_t file;  // File scope + program lifetime

- **Scope**: Only accessible within this (translation unit) **file**, file Scope
- **Lifetime**: Entire program lifetime
- **Linkage**: Internal linkage (hidden from other files)

### 2. **Function-Scope Static** (Inside Functions)

void function() {  
    static lpd_open_file_t file;  // Function scope + program lifetime  
}

- **Scope**: Only accessible within this **function**, function scope
- **Lifetime**: Entire program lifetime
- **Linkage**: No linkage (local to function)

### Static Functions in Headers

Short answer: **Yes, you _can_ put function definitions in headers — but only in very specific cases.**  
In normal C code, **headers should contain declarations, not definitions**, _except_ for a few well-defined patterns.

Let’s break it down precisely.

---

#### 1. The Default Rule (Most C Code)

**Headers (`.h`)**

```c
int foo(int x);   // declaration only
```

**Source files (`.c`)**

```c
int foo(int x) {
    return x * 2;
}
```

Why?  
Because headers are included in **multiple translation units**, and a normal (non-static) function definition would violate the **One Definition Rule** at link time.

---

#### 2. ❌ Non-static Function Definitions in Headers (Usually WRONG)

```c
// bad.h
int foo(int x) {
    return x * 2;
}
```

Included in:

```c
// a.c
#include "bad.h"

// b.c
#include "bad.h"
```

🔴 Result:

```
multiple definition of `foo`
```

Each `.c` file gets its own copy → linker error.

---

#### 3. ✅ `static` Functions in Headers (COMMON & SAFE)

```c
// util.h
static int foo(int x) {
    return x * 2;
}
```

Why this works:

- `static` gives **internal linkage**
    
- Each translation unit gets **its own private copy**
    
- No symbol conflicts
    

This is **very common** for:

- Helper functions
    
- Small utilities
    
- Inline-style helpers
    

#### NGINX-style example (relevant to your work):

```c
static ngx_inline int
ngx_min(int a, int b)
{
    return a < b ? a : b;
}
```

---

#### 4. ✅ `static inline` in Headers (BEST PRACTICE)

This is the **canonical modern C pattern**:

```c
// math_utils.h
static inline int add(int a, int b) {
    return a + b;
}
```

Benefits:

- No linker issues
    
- Compiler may inline
    
- Zero overhead helpers
    
- Header-only utilities
    

#### Why `inline` alone is NOT enough in C

In **C (not C++)**, `inline` does **not** imply internal linkage.

❌ Wrong:

```c
inline int add(int a, int b) {
    return a + b;
}
```

✔ Correct:

```c
static inline int add(int a, int b) {
    return a + b;
}
```

---

#### 5. ⚠️ `inline` Without `static` (Advanced / Rare)

This is legal but subtle:

```c
// header.h
inline int foo(int x) {
    return x * 2;
}
```

Then **exactly one** `.c` file must provide:

```c
int foo(int x);
```

This is fragile and rarely worth it.  
Most projects (including NGINX) **avoid this pattern entirely**.

---

#### 6. When You SHOULD Put Definitions in Headers

✔ Use headers for function definitions when:

- The function is **`static`**
    
- The function is **`static inline`**
    
- It’s a **macro-replacement helper**
    
- It’s performance-critical and tiny
    
- You want header-only utilities
    

✔ Examples:

- Bit-twiddling helpers
    
- String length helpers
    
- Flag checks
    
- Hot-path helpers in parsers / event loops
    

---

#### 7. When You SHOULD NOT

❌ Do NOT put function definitions in headers when:

- The function has external linkage
    
- The function is large
    
- The function is part of a public API
    
- You want a single global symbol
    

---

#### 8. Summary Table

|Pattern|Allowed in header?|Safe?|Notes|
|---|---|---|---|
|`int foo();`|✅|✅|Declaration only|
|`int foo() {}`|❌|❌|Multiple definition|
|`static int foo() {}`|✅|✅|One copy per TU|
|`static inline int foo()`|✅|✅|Best practice|
|`inline int foo()`|⚠️|⚠️|Requires external definition|

---

#### 9. Recommendation for _Your_ Server Project

Given your NGINX-style architecture:

- **Headers** → `static inline` helpers
    
- **.c files** → real logic, exported symbols
    
- Hot-path FSM helpers? → `static inline`
    
- Module APIs? → declarations only
    

If you want, I can:

- Audit one of your headers
    
- Show how NGINX organizes this pattern
    
- Help you design a header-only utility module safely

## when variables are declared locally on the stack inside a function they are unitialized and don't point anywhere, however when variables are declared outside functions or with static they are automatically initialized to 0

## Variadic Argument Indicator 

 `...`  = Variadic argument inicator states that this function can accept more arguements after the declared parameters -- and I don't know their exact number or type at compile time.

The purpose is to allow a `printf`-style message:

ngx_log_error_core(NGX_LOG_ERR, log, 0,
                   "Failed to open file \"%s\": %d", filename, errcode);
That corresponds to:

- `fmt = "Failed to open file \"%s\": %d"`
    
- The `...` contains:
    
    - A `const char *filename`
        
    - An `int errcode`



void my_log(const char *fmt, ...) {
    va_list args;             // 1. Declare a va_list variable
    va_start(args, fmt);      // 2. Initialize it to retrieve the variable arguments (after 'fmt')

    // Example: forward the args to vprintf
    vprintf(fmt, args);       // 3. Use it with a function that takes va_list

    va_end(args);             // 4. Clean up when done
}

## Const in Parameter Field Pattern

| Declaration          | Can reassign pointer? | Can change pointee? |
| -------------------- | --------------------- | ------------------- |
| `int* p`             | ✅ yes                 | ✅ yes               |
| `const int* p`       | ✅ yes                 | ❌ no                |
| `int* const p`       | ❌ no                  | ✅ yes               |
| `const int* const p` | ❌ no                  | ❌ no                |

## Extern 

- Extern is a storage specifier
- states that this variable/function external linkage and that it's definition is elsehere (usually another file) it does not allocate storage unless there is an intializer

## String Literals

### unsigned char is an integer type and can be a number: 

Short answer: **kind of — but with strict rules.**  
Long answer (this is one of those “C is honest, not friendly” moments):

---

#### String literal vs character literal (big distinction)

##### ✅ **Character literal** → **is a number**

```c
'a'   // type: int
```

- Value: implementation-defined, but **usually ASCII 97**
    
- This works:
    

```c
int x = 'a';     // x = 97
char c = 'a';    // c = 97 truncated to char
```

So **character literals are integers** in C.

---

#### ❌ **String literal** → **NOT a number**

```c
"a"
```

- Type: `char[2]` ( `'a'` + `'\0'` )
    
- When used in expressions, it **decays to `char *`**
    
- This is **not** a numeric value
    

```c
int x = "a";     // ❌ compile error
```

---

#### Why this confusion happens

They _look_ similar:

|Literal|Meaning|
|---|---|
|`'a'`|single character → integer|
|`"a"`|string → pointer to memory|

But internally:

```c
'a'   → 97
"a"   → { 'a', '\0' } stored in memory
```

---

#### When a string literal “acts like” a number

#### 1️⃣ Pointer context

```c
printf("%p\n", "a");   // prints an address
```

Here `"a"` decays to `char *`.

---

#### 2️⃣ Indexing into the string

```c
"a"[0] == 'a'    // true
"a"[1] == '\0'
```

This works because:

```c
"a"[0]  ≡  *("a" + 0)
```

So now you’re back to **character literals**, which _are_ integers.

---

#### Classic pitfall

```c
char c = "a";    // ❌ wrong
char c = 'a';    // ✅ correct
```

---

#### ASCII math example (why sliding windows work)

```c
int idx = 'a' - 'a';   // 0
int idx = 'z' - 'a';   // 25
```

But this:

```c
"a" - "a";  // ❌ illegal
```

---

#### TL;DR

| Literal  | Is it a number? | Why                              |
| -------- | --------------- | -------------------------------- |
| `'a'`    | ✅ yes           | character literal → `int`        |
| `"a"`    | ❌ no            | string literal → array → pointer |
| `"a"[0]` | ✅ yes           | yields `'a'`                     |

If you want, next we can talk about **why `'a'` is `int` instead of `char`**, or how this plays into parsing and tokenization (very relevant to your HTTP parser).

## Volatile
[[Volatile C]]