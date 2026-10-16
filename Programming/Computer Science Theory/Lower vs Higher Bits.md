Related to [[Computer Science]], [[Algorithm to Find Binary from Decimal Number]]
#computer_science #lower_vs_higher_bits

When I said **“lower bits”**, I mean the **least significant bits (LSBs)** — the bits on the **right side** of a binary number.

Binary positions look like this:

```
bit index:   7 6 5 4 3 2 1 0
binary:      b b b b b b b b
                          ↑
                    lowest bit
```

- **Lower bits** → right side → smaller powers of two
    
- **Higher bits** → left side → larger powers of two
    

Each bit position represents a power of two:

```
bit:   7   6   5   4   3   2   1   0
value:128 64  32  16  8   4   2   1
```

So for example:

```
13 = 00001101
```

```
higher bits        lower bits
0000               1101
```

The **lower bits are the ones representing smaller values** (1, 2, 4, 8, etc.).

---

## Why `& MASK` keeps the lower bits

Consider this mask used in ring buffers:

```
MASK = 7
```

Binary:

```
7 = 00000111
```

Now suppose:

```
index = 13
```

Binary:

```
13 = 00001101
```

Perform the bitwise AND:

```
   00001101
&  00000111
-----------
   00000101
```

Result:

```
5
```

What happened?

The mask:

```
00000111
```

**preserves only the lowest 3 bits**.

```
      higher bits removed
      ↓
0000 1101
    ↑↑↑
   lower bits
```

Those lower 3 bits are:

```
101 = 5
```

Which is exactly:

```
13 % 8 = 5
```

---

# Why this works for ring buffers


If your buffer size is a **power of two**:

```
size = 8
mask = size - 1 = 7
```

Then:

```
index % size
```

is the same as:

```
index & mask
```

because the mask **cuts off all higher bits and keeps only the bits needed to represent numbers 0–7**.

---

# Intuition

Think of the mask like **keeping only the last few binary digits**:

```
decimal: 29
binary:  11101

mask 7:  00111
```

```
11101
00111
-----
00101  → 5
```

You’re basically saying:

> “Ignore everything except the last 3 bits.”

---

# Why systems programmers love this

Instead of:

```c
index = (index + 1) % 1024;
```

you can do:

```c
index = (index + 1) & 1023;
```

which avoids **division**, making it **much faster** in tight loops like:

- ring buffers
    
- packet queues
    
- kernel schedulers
    
- networking stacks
    
- logging systems (like the one you built)
    

---

If you'd like, I can also show you **the visual trick NGINX and kernel developers think about when they design ring buffers using masks** — it makes this concept click instantly.