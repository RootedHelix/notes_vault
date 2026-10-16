
## Function Declaration and Definition
A function consist of two parts:

- **Declaration:** the function's name, return type, and parameters (if any)
- **Definition:** the body of the function (code to be executed)

void **myFunction()** { // **declaration**  
  // the body of the function (**definition**)  
}

## Function Prototypes

A **function prototype** in C is a **declaration** of a function that tells the compiler its name, return type, and parameters **before its actual definition**. It acts as a forward declaration, allowing functions to be used before they are defined.

---

### **🚀 Basic Syntax**

```c
return_type function_name(parameter_type1, parameter_type2, ...);
```

---

### **✅ Example Without a Prototype (Incorrect in Some Cases)**

```c
#include <stdio.h>

int main() {
    hello();  // ❌ Compiler error: 'hello' is undeclared
    return 0;
}

void hello() {
    printf("Hello, World!\n");
}
```

💡 **Problem:** The compiler doesn't know about `hello()` when it's first encountered in `main()`.

---

### **✅ Example With a Prototype (Correct)**

```c
#include <stdio.h>

// Function prototype (declaration)
void hello();

int main() {
    hello();  // ✅ No error, compiler knows about hello()
    return 0;
}

// Function definition
void hello() {
    printf("Hello, World!\n");
}
```

💡 **Fix:** The prototype `void hello();` tells the compiler that `hello()` exists.

---

### **🔹 Why Use Function Prototypes?**

✅ **Allows functions to be called before they are defined.**  
✅ **Helps catch errors early** (e.g., mismatched return types or parameters).  
✅ **Supports modular code** (e.g., when functions are declared in `.h` files).

---

### **✅ Example With Parameters**

```c
#include <stdio.h>

// Function prototype
int add(int a, int b);

int main() {
    int sum = add(5, 3);
    printf("Sum: %d\n", sum);
    return 0;
}

// Function definition
int add(int a, int b) {
    return a + b;
}
```

✔ The prototype `int add(int, int);` tells the compiler what to expect.

---

### **🔹 When Are Prototypes Required?**

- When defining functions **after** `main()`.
- When using **multiple source files** (`.c` and `.h` files).
