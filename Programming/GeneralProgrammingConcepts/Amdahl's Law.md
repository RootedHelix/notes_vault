Amdahl’s Law is the **hard reality check** for parallelization. It tells you _exactly_ why some programs scale beautifully with more cores… and why others hit a wall no matter how many CPUs you throw at them.

Let’s explain it in a way that maps **directly to your backtest system**.

---

## The one-sentence definition

> **Amdahl’s Law says:**  
> _Your maximum speedup from parallelism is limited by the part of the program that cannot be parallelized._

No exceptions. Ever.

---

## The formula (simple, not scary)

[  
\text{Speedup}(N) = \frac{1}{(1 - P) + \frac{P}{N}}  
]

Where:

- **P** = fraction of the program that _can_ be parallelized
    
- **N** = number of cores/threads
    
- **(1 − P)** = serial portion (the bottleneck)
    

---

## Why this matters more than “number of cores”

Let’s plug in numbers that look like your code.

### Suppose your runtime is:

|Part|% of time|
|---|---|
|Python setup / I/O|5%|
|Model inference|10%|
|Backtest kernel|**85%**|

So:

- **P = 0.85**
    
- Serial fraction = 0.15
    

---

## What happens as you add cores?

### With 2 cores

[  
\frac{1}{0.15 + 0.85/2} ≈ 1.74×  
]

### With 4 cores

[  
\frac{1}{0.15 + 0.85/4} ≈ 2.76×  
]

### With 8 cores

[  
\frac{1}{0.15 + 0.85/8} ≈ 4.3×  
]

### With ∞ cores (the theoretical limit)

[  
\frac{1}{0.15} ≈ 6.7×  
]

👉 **No matter how many cores you add, you can never exceed ~6.7×.**

That’s Amdahl’s Law in action.

---

## The killer insight (this is the important part)

> **Optimizing the serial portion matters more than adding cores.**

If you reduce the serial part from 15% → 5%:

[  
\text{Max speedup} = \frac{1}{0.05} = 20×  
]

Same machine. Same cores.  
Just better structure.

---

## How this maps _exactly_ to your backtest

### Serial today

- Python orchestration
    
- Asset dispatch loop
    
- Per-asset kernel calls
    
- CSV writing
    

### Parallelizable

- Asset backtests
    
- Multiple runs
    
- Parameter sweeps
    

Your goal is to:

- Push **as much time as possible** into the parallel region
    
- Shrink the serial fraction
    

That’s why we talked about:

- C-level threading
    
- Batch execution
    
- Single entrypoint kernels
    

Each one **increases P**.

---

## Why people misapply Amdahl’s Law

Common mistake:

> “Parallelism doesn’t scale past X cores.”

Wrong.

Correct interpretation:

> “You parallelized the wrong thing.”

If you parallelize:

- Inner loops with dependencies → low P
    
- Independent tasks → high P
    

Your workload is the _good_ kind.

---

## Amdahl vs Gustafson (quick aside)

- **Amdahl**: fixed problem size
    
- **Gustafson**: problem size grows with cores
    

In quant research:

- You _always_ grow problem size (more assets, more scenarios)
    
- So real-world scaling is often _better_ than Amdahl predicts
    

But Amdahl still defines the **upper bound per run**.

---

## One mental model to remember

Think of your program as:

```text
[ SERIAL ][ PARALLEL ][ SERIAL ]
```

No matter how fast the middle gets:

- The ends cap total speed
    

Your job as an optimizer is to:

> **Shave the ends, widen the middle**

---

## Bottom line

- Amdahl’s Law is not pessimism — it’s a **diagnostic tool**
    
- It tells you _where_ optimization matters
    
- In your case:
    
    - You already optimized the core
        
    - Next step is maximizing **parallel fraction**
        

If you want, we can:

- Estimate **P** from your real timings
    
- Predict your exact speedup curve
    
- Decide whether asset-level or run-level parallelism wins for you
    

This is exactly the right point to be thinking about Amdahl’s Law.