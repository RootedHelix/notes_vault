Related to [[Computer Science]], [[C General Concepts]]

#computer_science #binary_algorithm 

The classic algorithm to convert a **decimal (base-10) integer to binary (base-2)** is the **repeated division by 2 algorithm**.

---

## Algorithm: Repeated Division by 2

1. Take the decimal number **N**.
    
2. Divide **N by 2**.
    
3. Record the **remainder** (0 or 1).
    
4. Replace **N** with the **quotient**.
    
5. Repeat steps 2–4 until **N = 0**.
    
6. The **binary number is the remainders read from bottom to top**.
    

---

## Example: Convert 13 → Binary

|Step|N ÷ 2|Quotient|Remainder|
|---|---|---|---|
|1|13 ÷ 2|6|1|
|2|6 ÷ 2|3|0|
|3|3 ÷ 2|1|1|
|4|1 ÷ 2|0|1|

Now read **remainders from bottom to top**:

```
1101
```

So:

```
13₁₀ = 1101₂
```

---

## Why This Works

Binary is a **base-2 positional system**:

[  
N = b_0·2^0 + b_1·2^1 + b_2·2^2 + ...  
]

Each division extracts the **least significant bit**:

```
N % 2 → bit 0
(N/2) % 2 → bit 1
(N/4) % 2 → bit 2
```

So the algorithm is literally **extracting the bits one by one**.

---

## How This Looks in Code (C)

```c
void decimal_to_binary(unsigned int n) {
    int bits[32];
    int i = 0;

    while (n > 0) {
        bits[i++] = n % 2;
        n /= 2;
    }

    for (int j = i - 1; j >= 0; j--) {
        printf("%d", bits[j]);
    }
}
```

---

## Even Faster Way (Bitwise Version)

Since you're working with **low-level systems and bitwise operations** (like your ring buffer mask earlier), the **actual hardware-level approach** is:

```
bit = n & 1
n >>= 1
```

Example loop:

```c
while (n > 0) {
    printf("%d", n & 1);
    n >>= 1;
}
```

This extracts the **least significant bit directly**.

---

## Important Connection to Your Ring Buffer Question

When you saw this:

```
(rb->head + 1) & MASK
```

That works because when **MASK = size - 1** and size is a **power of two**, the mask acts like a **binary modulo operator**.

Example:

```
index % 8   ==   index & 7
```

Because:

```
8  = 1000
7  = 0111
```

So the mask keeps only the **lower bits**.

---

✅ If you'd like, I can also show you the **much deeper computer-science interpretation** of binary conversion as **greedy decomposition into powers of two**, which is how compilers and CPUs actually reason about integers. It's a neat connection to systems programming.