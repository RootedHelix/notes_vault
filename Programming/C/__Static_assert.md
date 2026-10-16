Related to [[C]],
#C #__Static_assert

> Compile Time assertion

## AI Response

`__static_assert` is not a standard C or C++ keyword. It’s almost certainly:

1. A **typo**, or
    
2. A **compiler-/codebase-specific macro or internal symbol**
    

What you’re likely looking for is:

### ✅ Standard form (C11 and later)

Use static_assert:

```c
static_assert(sizeof(int) == 4, "int must be 4 bytes");
```

### Or the underlying keyword:

```c
_Static_assert(sizeof(int) == 4, "int must be 4 bytes");
```

---

## What it does

A static assertion is evaluated **at compile time**, not runtime.

- If the condition is **true** → compilation continues
    
- If **false** → compilation fails with the provided message
    

This is critical in embedded systems where assumptions about:

- struct layout
    
- register sizes
    
- alignment  
    must be enforced deterministically
    

---

## Why you might see `__static_assert`

Common scenarios:

### 1. Internal compiler macro

Compilers like GCC or Clang sometimes define internal helpers using `__` prefixes.

### 2. Compatibility shim

Older codebases (pre-C11) sometimes define:

```c
#define __static_assert(cond, msg) _Static_assert(cond, msg)
```

or worse, a hack like:

```c
#define __static_assert(cond, msg) typedef char static_assertion[(cond)?1:-1]
```

### 3. You misread `_Static_assert`

Very common visually—especially in embedded headers.

---

## Practical takeaway (for your context)

In your embedded work (STM32, HAL, DMA, etc.), you should:

- Prefer `_Static_assert` (portable C11)
    
- Use it for:
    
    - buffer sizes
        
    - struct packing
        
    - ring buffer constraints (e.g., power-of-two size)
        

Example relevant to your ring buffer:

```c
_Static_assert((HLX_RING_BUFFER_SIZE & (HLX_RING_BUFFER_SIZE - 1)) == 0,
               "Ring buffer size must be power of 2");
```

---

If you want, paste where you saw `__static_assert`—I can tell you exactly what that specific implementation is doing.