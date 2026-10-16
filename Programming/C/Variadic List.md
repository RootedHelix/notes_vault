Related to [[C]], [[<stdarg.h>]]

The `<stdarg.h>` library lets you write functions that let you accept a variable number of arguments like: 

```C
printf("value: %d\n", x);
```

They let you write functions that accept a variable number of arguments, like:

```
printf("value: %d\n", x);
```

The function does not know at compile time how many extra arguments it will receive. The `stdarg.h` macros give you a controlled way to walk through those arguments.

---

A variadic function has at least one normal parameter, then `...`.

```C
int append_to_form(json_form_t* analysis, const char* format, ...)
```

## The four main pieces

### 1. `va_list`

```
va_list args;
```

`va_list` is a special type that stores the state needed to access the unnamed arguments.

Conceptually, you can think of it as a cursor pointing at the next variadic argument.

It is not necessarily a simple pointer. On modern systems, arguments may be passed through registers, stack memory, or both, depending on the ABI. `va_list` hides those platform details.

---

### 2. `va_start`

```
va_start(args, format);
```

`va_start` initializes the `va_list`.

The second argument must be the **last named parameter before the `...`**.

In this function:

```
int analysis_append(file_analysis_t *analysis, const char *format, ...)
```

the last named parameter is:

```
format
```

So you write:

```
va_start(args, format);
```

That tells C:

> Start reading the anonymous arguments that come after `format`.

Example:

```
analysis_append(&a, "line: %u\n", 120);
```

Inside the function:

```
analysis = &aformat   = "line: %u\n"...      = 120
```

After this:

```
va_start(args, format);
```

`args` is now ready to read `120`.

---

### 3. `va_arg`

```
int value = va_arg(args, int);
```

`va_arg` retrieves the next variadic argument and advances the internal cursor.

Example:

```
void example(const char *label, ...){
    va_list args;    
    va_start(args, label);    
    int number = va_arg(args, int);    
    va_end(args);
    }
```

Call:

```
example("age", 25);
```

Inside:

```
int number = va_arg(args, int);
```

gets:

```
25
```

If there are multiple arguments:

```C
void example(const char *label, ...){
    va_list args;    
    va_start(args, label);    
    int number = va_arg(args, int);    
    const char *text = va_arg(args, const char *);    
    va_end(args);
    }
```

Call:

```
example("data", 42, "hello");
```

Then:

```
number = 42text   = "hello"
```

The order must match exactly.

---

### 4. `va_end`

```
va_end(args);
```

`va_end` cleans up the `va_list`.

Always call it before returning from the function after `va_start`.

Even if on many systems it appears to do nothing, the C standard requires it. Some platforms may need cleanup.