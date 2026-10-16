In **CPython**, yes — **all variables are references (pointers) to objects**, but there’s an important precision to that statement that’s worth getting exactly right.

---

## The precise statement

> **Python variables do not store values.  
> They store references to objects.**

Those references are implemented as **C pointers (`PyObject *`)**.

So when you write:

```python
x = 10
```

What happens conceptually is:

```
x ──▶ PyLongObject(10)
```

`x` is not “10”.  
`x` is a pointer to an object whose value is 10.

---

## What is _actually_ a pointer?

### Variables / locals / names

- Local variables
    
- Function arguments
    
- Global variables
    
- Attributes
    
- List elements
    
- Dict values
    

All of these store **references** (pointers) to objects.

### Objects

- Live on the heap
    
- Have identity
    
- Have a type
    
- Have a reference count
    

---

## What is _not_ a pointer?

Inside some objects, data may be stored **by value**:

### Example: NumPy array

```python
arr = np.array([1.0, 2.0, 3.0])
```

- `arr` → pointer to `np.ndarray` object
    
- `arr.data` → contiguous C buffer of `double`
    

The buffer elements are **not Python objects** and **not pointers**.

This distinction is everything for performance.

---

## Why people get confused

Python _looks_ like this:

```python
a = 5
b = a
b += 1
```

But what actually happens is:

```
a ──▶ PyLongObject(5)
b ──▶ PyLongObject(5)

b += 1
   └─▶ new PyLongObject(6)
b now points to 6
a still points to 5
```

There was never a “variable holding a value”.

---

## Why Python must work this way

Because Python supports:

- dynamic typing
    
- polymorphism
    
- aliasing
    
- object identity (`is`)
    
- mutable objects
    

Example:

```python
lst = []
a = lst
b = lst
b.append(1)
```

If variables weren’t references, this would be impossible.

---

## Stack vs heap clarification

Python **does have a stack**, but it holds:

- pointers to objects
    
- frame state
    
- temporary references
    

It does **not** hold raw integers, floats, structs, or arrays.

---

## Comparison with C (important for your optimization work)

|Concept|Python|C|
|---|---|---|
|Variable stores|Pointer|Value|
|Integer storage|Heap object|Stack/register|
|Passing argument|Reference|Value (unless pointer)|
|Assignment|Rebind pointer|Copy value|

This is why translating Python loops directly to C without changing data layout often disappoints.

---

## One subtle exception (don’t overinterpret it)

Some implementations (PyPy, JITs) _may_ optimize scalars internally.

But:

- You are using CPython
    
- C extensions and NumPy assume pointer semantics
    
- You should program as if everything is a pointer
    

---

## The mental model you want

Think of Python as:

> “A language where every variable is a tagged pointer, and performance comes from minimizing pointer chasing.”

Once you internalize that, things like:

- flat arrays
    
- NumPy
    
- C extensions
    
- avoiding dicts in hot loops
    

all click into place.

---

If you want next, I can:

- draw a memory diagram of your backtest loop
    
- show how many pointer dereferences happen per iteration
    
- or show how your loop would look _exactly_ in C with equivalent semantics
    

Just say the word.