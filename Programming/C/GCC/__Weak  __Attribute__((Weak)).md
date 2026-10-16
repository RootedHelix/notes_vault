
Related to [[GCC]], [[GCC Compiler Features]]
#gcc #__weak 

`__weak` is a Compiler Feature of GCC

**STM32 just defines**:

`#define __weak __attribute__((weak))`

So `__weak` is basically a convenience macro.

## What does __Weak do

`__weak` makes a function or variable **optional and overridable** at link time.

The simple definition:

> A weak symbol can be replaced by a normal (strong) symbol with the same name.

---

# 🔹 What It Actually Does

When you declare something as weak:

```c
__weak void foo(void)
{
    // default behavior
}
```

You are telling the linker:

> “Use this only if nobody else provides a stronger version.”

---

# 🔹 Example (Exactly How STM32 Uses It)

In the HAL:

```c
__weak void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    // Default: do nothing
}
```

In your code:

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
}
```

At link time:

- HAL version = weak
    
- Your version = strong
    
- Linker chooses yours
    

No errors. No warnings. Automatic override.

---

# 🔹 What Happens in Different Cases

### Case 1: Only weak exists

✔ Weak function is used.

### Case 2: Weak + strong exist

✔ Strong one overrides weak.

### Case 3: Two strong definitions

❌ Linker error (multiple definition).

### Case 4: Two weak definitions

⚠ One is chosen (usually first found).

---

# 🔹 Important: This Is a Linker Behavior

`__weak` does NOT:

- Change how the function runs
    
- Add runtime overhead
    
- Add virtual dispatch
    
- Use function pointers
    

It is resolved **before the program runs**, during linking.

---

# 🔹 What It's Used For

Common uses in embedded:

- Default interrupt handlers
    
- Default callbacks
    
- Library hooks
    
- Startup code handlers
    
- Customizable low-level init functions
    

---

# 🔹 Where It Comes From

In GCC:

```c
__attribute__((weak))
```

STM32 just defines:

```c
#define __weak __attribute__((weak))
```

So it’s a compiler/linker feature — not standard C.

---

# 🧠 One-Sentence Mental Model

`__weak` = “This definition is replaceable if a stronger one exists.”

---

If you'd like, I can show you a tiny two-file example that demonstrates exactly how the linker chooses between weak and strong symbols.