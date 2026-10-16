Related to [[C]], 
#C #function_signature

# AI Response

A **function signature** is the portion of a function’s definition that uniquely identifies how the function can be called.

### Core definition (language-agnostic)

A function signature consists of:

- **Function name**
    
- **Parameter types (and order)**
    
- _(Sometimes)_ **parameter count**
    

### What it does _not_ include

- The function body
    
- Parameter names (in many languages, names are irrelevant)
    
- Return type _(depends on language — see below)_
    

---

## In C (your context)

In C, the function signature is typically considered:

- **Function name + parameter types**
    

Example:

```c
int add(int a, int b);
```

Signature:

```c
add(int, int)
```

Notes:

- Parameter _names_ (`a`, `b`) are not part of the signature
    
- Return type (`int`) is **not used for overloading**, so it’s usually not considered part of the signature in practice
    

---

## Why it matters

The signature defines:

- How the compiler checks **correct usage**
    
- Whether two functions are considered the “same” or not
    

In C specifically:

- You **cannot overload functions**, so:
    

```c
int add(int, int);
float add(float, float); // ❌ illegal in C
```

---

## Contrast with C++ (important distinction)

In C++:

- Function signatures include parameter types
    
- **Function overloading is allowed**
    

```cpp
int add(int, int);
float add(float, float); // ✅ valid (different signatures)
```

Return type alone still does **not** differentiate signatures.

---

## Quick mental model

Think of the function signature as:

> “The contract required to call this function correctly.”

---

If you want, I can connect this to function pointers or ABI-level calling conventions, which is where this concept becomes more critical in embedded systems.