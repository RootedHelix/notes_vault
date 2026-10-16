Related to [[C]], [[C General Concepts]]
#volatile_c


> **“Volatile prevents the compiler from optimizing away or caching accesses to a variable whose value may change outside normal program flow.”**

>`volatile` solves **visibility problems**, not **synchronization problems**.
## What `volatile` Actually Guarantees

When you declare:

`volatile uint32_t msTicks;`

You are telling the compiler:

> “This memory location may change at any time outside your visible code flow.”

So the compiler must:

- Perform every read exactly as written
    
- Perform every write exactly as written
    
- Not cache it in a register across accesses
    
- Not remove or merge accesses
    
- Not assume it stays constant
    

So yes — **each access must result in a real memory read/write.**

---

## What It Does _Not_ Mean

It does NOT mean:

- It forces physical RAM access (it might still be in cache on higher-end systems)
    
- It makes operations atomic
    
- It prevents race conditions
    
- It creates thread safety
    
- It inserts memory barriers
    

It only controls **compiler behavior**, not CPU hardware behavior.

On Cortex-M4 (like your STM32F446RE), there is no data cache anyway, so memory reads are straightforward.

---

## Why This Matters in Your Case

Without `volatile`:

`while (1) {     if (msTicks >= 1000)`

The compiler could transform this to:

`uint32_t temp = msTicks;  while (1) {     if (temp >= 1000)`

Because it sees no code modifying `msTicks` in that scope.

But the interrupt modifies it.

So now the compiler’s optimization becomes incorrect relative to reality.

---

## What Changes in Assembly

Without `volatile`, compiler might do:

```Assembly
LDR R4, =msTicks   ; load once loop:
CMP R4, #1000
B ...
          
```


With `volatile`:

```Assembly
loop:     LDR R4, =msTicks   ; load every iteration
CMP R4, #1000     B ...
```

That’s the difference.

## Volatile in Relation to Embedded
#volatile_embedded
In embedded systems there are three "worlds":

1. The compiler world
    
2. The CPU execution world
    
3. The hardware/interrupt world
    

`volatile` tells the compiler:

> “The hardware world exists. Don’t optimize as if it doesn’t.”

