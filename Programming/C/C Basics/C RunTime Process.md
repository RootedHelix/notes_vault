Related to [[C]]


The **runtime process** in C refers to the sequence of activities and mechanisms that occur when a C program is executed. It involves the interaction between the program and the operating system, as well as the management of program execution by the runtime environment. Here’s an overview of the key components and steps:

---

### 1. **Compilation and Linking** (Before Runtime)
Before a program can run, it must go through:
- **Compilation**: The C code is translated into machine code by the compiler.
- **Linking**: External libraries and object files are linked to produce an executable.

At runtime, the executable file is loaded into memory for execution.

---

### 2. **Loading and Execution Start**
When you execute a C program:
1. **Program Loading**:
   - The operating system loads the program into memory.
   - The program's **text segment** (code), **data segment** (global/static variables), and **stack** are allocated.
   
2. **Entry Point**:
   - The program begins execution from the `main()` function.

---

### 3. **Memory Management** -- Process Memory Layout 
#process_memory_layout

Memory for a C program is divided into the following segments:
- **Code Segment — `.text`**: compiled machine code (**R-X** at runtime; readable + executable, not writable).
    
- **Data (static) storage**
    
    - **Initialized, writable — `.data`**: globals/statics with explicit **non-zero** initializers.
        
    - **Uninitialized / zero-init — `.bss`**: globals/statics **without** an initializer or explicitly initialized to 0 (occupies no bytes in the file; **zeroed by the loader**).
        
    - **Read-only initialized — `.rodata`**: string literals and `const` tables (**read-only** at runtime).
        
- **Heap**: runtime memory from `malloc`/`calloc`/`realloc` (RW, non-executable); not a section.
    
- **Stack**: per-thread call frames and automatic locals (RW, non-executable); created by the OS at runtime; not a section.


Sections (file)              →   Segments (runtime)
.text (code)                 →   RX load segment
.rodata (const)              →   R  load segment
.data (init RW), .bss (zero) →   RW load segment  ← often called “data segment”
(stack, heap)                →   created at runtime; not file sections

High addresses
┌───────────────┐
│     Stack                  │
├───────────────┤
│  mmap region        │
│ shared libs             │
├───────────────┤
│     Heap                  │
├───────────────┤
│      BSS                   │
├───────────────┤
│      Data                   │
├───────────────┤
│     Rodata                │
├───────────────┤
│      Text                   │
└───────────────┘
Low addresses


---

### 4. **Runtime Support**
The C runtime includes:
- **Startup Code**: A small piece of code provided by the runtime environment that runs before `main()`. It:
  - Sets up the stack and heap.
  - Initializes global/static variables.
  - Prepares command-line arguments.
- **Library Functions**:
  - Functions like `printf`, `scanf`, `malloc`, etc., are part of the C standard library and are loaded and executed during runtime.

---

### 5. **Execution Flow**
- Functions and statements in the program execute sequentially, calling library functions or system calls as needed.
- The stack tracks function calls, return addresses, and local variables.

---

### 6. **Error Handling**
The runtime environment manages errors such as:
- **Segmentation Faults**: Accessing invalid memory.
- **Division by Zero**: Undefined behavior.
- **Runtime Exceptions**: Signals like `SIGSEGV`, `SIGFPE`, etc.

---

### 7. **Program Termination**
- The program terminates when:
  - `main()` finishes.
  - An exit function like `exit()` is called.
- Cleanup:
  - The runtime environment deallocates resources like file handles and memory.

---

### Summary
The runtime process in C involves:
1. Loading the program into memory.
2. Executing the `main()` function.
3. Managing memory, function calls, and library interactions.
4. Terminating and cleaning up resources.

While C has a minimal runtime compared to languages like Java or Python (which have extensive virtual machines), it relies on the operating system and standard libraries to provide necessary runtime functionality.