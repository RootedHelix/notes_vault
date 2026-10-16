To use `PyObject` **in C**, you must include the **CPython C API header**.

### The short, correct answer

```c
#include <Python.h>
```

That’s it.  
`Python.h` pulls in **everything** you are allowed to use from the Python runtime, including `PyObject`.

---

### Why `Python.h` is required (and nothing else is)

`PyObject` is defined deep inside CPython headers, but you **must not** include those directly.

Bad (never do this):

```c
#include <object.h>      // ❌ internal
#include <pyobject.h>   // ❌ internal
```

Good (always do this):

```c
#include <Python.h>     // ✅ public, stable API
```

`Python.h` handles:

- `PyObject`
    
- Reference counting macros
    
- Type checking
    
- Method calls
    
- Buffer protocol
    
- Module initialization
    
- ABI compatibility
    

It also enforces the correct compile-time flags.

---

### Minimal example that uses `PyObject`

```c
#include <Python.h>

int is_weekend(PyObject *ts) {
    // placeholder — not recommended for hot loops
    return 0;
}
```

This will compile **only** if Python development headers are installed.

---

### Important: this only applies to Python extensions

You need `Python.h` **only if**:

- You are writing a CPython extension
    
- You are passing `PyObject*`
    
- You are embedding Python
    
- You are calling Python APIs from C
    

You **do not** need this if:

- Your C code operates purely on `double*`, `uint8_t*`, etc
    
- You expose your C function via:
    
    - `ctypes`
        
    - `cffi`
        
    - a raw shared library with no Python API usage
        

In your backtest design, the **hot loop should not include `Python.h` at all**.

---

### This distinction matters for your architecture

Correct layering:

```
Python
 ├── pandas / numpy
 ├── preprocessing (weekday, signals, masks)
 ├── marshaling to arrays
 └── calls into C

C (hot loop)
 ├── double*
 ├── uint8_t*
 └── tight loop (no Python.h)
```

Incorrect layering:

```
C hot loop
 ├── PyObject*
 ├── Python.h
 ├── method calls
 └── refcounting
```

---

### One-sentence rule you should keep

> If a `.c` file includes `Python.h`, it should **never** contain your performance-critical loop.

If you want, next we can:

- Decide whether your C core should be a Python extension or a pure shared lib
    
- Show a minimal extension _wrapper_ that isolates `Python.h`
    
- Or walk through how NumPy avoids `PyObject*` in inner loops



# Integrating the C hotloop into Python

Got it — you want **how to call this C kernel from Python**, not how to run it as a standalone C program.

At this stage, the **correct and minimal way** to do this is:

> **Compile the C kernel as a shared library and call it from Python using `ctypes` (or `cffi`) with NumPy arrays.**

I’ll show you the _clean, production-grade way_ that matches the architecture you already built.

---

## 1. What the Python↔C boundary should look like

Your C kernel is already perfect for Python interop because it:

- Uses **plain C types**
    
- Uses **contiguous arrays**
    
- Does **no allocation**
    
- Has **no Python awareness**
    

That is exactly what Python wants.

The contract becomes:

```
Python owns memory
↓
Python passes raw pointers
↓
C mutates output buffers
↓
Python reads results
```

No copying. No glue logic in the hot loop.

---

## 2. Compile the kernel as a shared library

### Build command (Linux / macOS)

```bash
gcc -O3 -fPIC -shared \
    src/backtest.c src/trade_state.c src/sizing.c src/curve.c \
    -Iinclude -o libbacktest.so -lm
```

(macOS: use `libbacktest.dylib` instead of `.so`)

You now have:

```
libbacktest.so
```

---

## 3. Python-side: define the ABI with `ctypes`

### Step 1 — load the library

```python
import ctypes
import numpy as np

lib = ctypes.CDLL("./libbacktest.so")
```

---

### Step 2 — define argument types exactly

This is **critical**. Your kernel signature maps 1:1.

```python
lib.run_backtest_core.argtypes = [
    ctypes.POINTER(ctypes.c_double),   # prices
    ctypes.POINTER(ctypes.c_double),   # preds
    ctypes.POINTER(ctypes.c_uint8),    # is_weekend
    ctypes.POINTER(ctypes.c_uint8),    # is_crypto
    ctypes.c_int,                      # n

    ctypes.c_double,                   # band
    ctypes.c_double,                   # flip_band
    ctypes.c_double,                   # dollars_per_trade
    ctypes.c_double,                   # conf_scale
    ctypes.c_double,                   # conf_full
    ctypes.c_int,                      # cooldown_bars
    ctypes.c_int,                      # reentry_confirm_bars
    ctypes.c_int,                      # flip_confirm_bars
    ctypes.c_int,                      # exit_on_flip_hold

    ctypes.c_int,                      # size_rule_enable
    ctypes.c_double, ctypes.c_double, ctypes.c_double,
    ctypes.c_double, ctypes.c_double, ctypes.c_double,

    ctypes.POINTER(ctypes.c_double),   # realized
    ctypes.POINTER(ctypes.c_int),      # pos_out
    ctypes.POINTER(ctypes.c_int),      # sig_out
    ctypes.POINTER(ctypes.c_double),   # qty_out
    ctypes.POINTER(ctypes.c_double),   # size_mult_out

    ctypes.c_void_p                    # curve_t*
]
```

Return type:

```python
lib.run_backtest_core.restype = None
```

---

## 4. Prepare NumPy arrays (this matters)

### Inputs must be:

- contiguous
    
- correct dtype
    
- writable (for outputs)
    

```python
n = len(preds)

prices = np.ascontiguousarray(prices, dtype=np.float64)
preds  = np.ascontiguousarray(preds,  dtype=np.float64)

is_weekend = np.ascontiguousarray(is_weekend, dtype=np.uint8)
is_crypto  = np.ascontiguousarray(is_crypto,  dtype=np.uint8)

realized   = np.zeros(n, dtype=np.float64)
pos_out    = np.zeros(n, dtype=np.int32)
sig_out    = np.zeros(n, dtype=np.int32)
qty_out    = np.zeros(n, dtype=np.float64)
size_mult  = np.zeros(n, dtype=np.float64)
```

---

## 5. Pass pointers safely

NumPy gives you raw pointers without copying:

```python
lib.run_backtest_core(
    prices.ctypes.data_as(ctypes.POINTER(ctypes.c_double)),
    preds.ctypes.data_as(ctypes.POINTER(ctypes.c_double)),
    is_weekend.ctypes.data_as(ctypes.POINTER(ctypes.c_uint8)),
    is_crypto.ctypes.data_as(ctypes.POINTER(ctypes.c_uint8)),
    n,

    band,
    flip_band,
    dollars_per_trade,
    conf_scale,
    conf_full,
    cooldown_bars,
    reentry_confirm_bars,
    flip_confirm_bars,
    exit_on_flip_hold,

    size_rule_enable,
    size_floor,
    size_mid,
    size_top,
    size_min_mult,
    size_mid_mult,
    size_max_mult,

    realized.ctypes.data_as(ctypes.POINTER(ctypes.c_double)),
    pos_out.ctypes.data_as(ctypes.POINTER(ctypes.c_int)),
    sig_out.ctypes.data_as(ctypes.POINTER(ctypes.c_int)),
    qty_out.ctypes.data_as(ctypes.POINTER(ctypes.c_double)),
    size_mult.ctypes.data_as(ctypes.POINTER(ctypes.c_double)),

    curve_ptr   # explained below
)
```

---

## 6. Handling `curve_t*` correctly

You have **two clean options**.

### Option A — create curve in C, expose a factory (recommended)

In C:

```c
curve_t* curve_create_default(void);
void curve_destroy(curve_t*);
```

Python:

```python
lib.curve_create_default.restype = ctypes.c_void_p
curve_ptr = lib.curve_create_default()
```

Later:

```python
lib.curve_destroy(curve_ptr)
```

This keeps Python **completely unaware** of the struct layout (best practice).

---

### Option B — pass `NULL` if unused

If sizing curves are optional:

```python
curve_ptr = None
```

and guard in C.

---

## 7. Reconstruct omitted metrics in Python (important)

Now that the kernel is done, you **recreate analytics cleanly**:

```python
entry_mask = (pos_out[:-1] == 0) & (pos_out[1:] != 0)
entry_confs = size_mult[1:][entry_mask]
entry_count = entry_mask.sum()
```

This exactly reproduces:

```python
entry_count += 1
entry_confs.append(size_mult)
```

without polluting the kernel.

---

## 8. Why this approach is the right one

- No Python in the hot loop
    
- No C memory ownership confusion
    
- No dynamic allocation in C
    
- Zero-copy NumPy ↔ C
    
- Kernel usable from **any language**
    

This is **how real quant engines are built**.

---

## 9. Mental rule (keep this)

> **Python orchestrates.  
> C computes.  
> Nothing else crosses the boundary.**

You implemented that perfectly.

---

If you want next, we can:

- switch from `ctypes` → `cffi` (cleaner, safer)
    
- build a tiny Python wrapper class
    
- add batch / multi-asset execution
    
- or design a versioned ABI so you never break callers
    

Just say the word.