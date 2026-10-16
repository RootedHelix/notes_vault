You’re very close, but there’s an important correction to make.  
What you just said is **almost right in English**, but **not right in systems terms** — and that distinction is _exactly_ why this conversation matters.

Let me be precise.

---

## The core correction

> **An invariant is _not_ an assumption.**  
> **An invariant is a condition that has been _proven_ and is _actively maintained_.**

An **assumption** is something you _hope_ is true.  
An **invariant** is something your code _guarantees_ is true.

That difference is the boundary between safe low-level C and fragile C.

---

## Formal definition (systems / PL sense)

An **invariant** is a property that:

1. Is established at a specific point
    
2. Is preserved by all code that can mutate the state
    
3. Can be relied upon without re-checking
    

If any code path can violate it, it is **not an invariant**.

---

## Concrete example (C, not philosophy)

### ❌ Assumption

```c
// assume buffer is large enough
memcpy(dst, src, len);
```

Nothing enforces this.  
This is hope, not an invariant.

---

### ✅ Invariant

```c
if (len > dst_capacity) return ERROR;
// from here on, len <= dst_capacity is an invariant
memcpy(dst, src, len);
```

Now:

- The condition was _checked_
    
- The function enforces it
    
- All following code can rely on it
    

This is how NGINX thinks.

---

## How invariants propagate across layers

### Low layer establishes invariants

Example:

- “This buffer is valid UTF-8”
    
- “This pointer range is inside allocated memory”
    
- “This HTTP request line is complete”
    

These are **actively proven** by parsing code.

---

### Higher layer consumes invariants

Example:

- “`r->uri` is a valid string”
    
- “`price` is a valid double”
    
- “This connection has a lifetime”
    

These are not re-checked — because they were already enforced.

---

## Why this matters for your original question

You were asking:

> “Why wouldn’t I just use pointer tuples everywhere like Python internals or NGINX core?”

Because **pointer arithmetic requires _local proof_**.

If you use raw pointers:

- You must prove bounds
    
- You must prove lifetime
    
- You must prove type correctness
    
- You must prove ownership
    

Every time.

If you don’t, you’ve made assumptions — not invariants.

---

## NGINX’s golden rule (unstated but real)

> **Pointer arithmetic is only allowed where invariants are fully established _inside the same function or module_.**

That’s why:

- Parsing code is paranoid
    
- State structs are boring
    
- Interfaces are typed
    

NGINX _never_ relies on “caller promises” for raw memory.

---

## Why structs encode invariants

A struct:

```c
typedef struct {
    double price;
    double qty;
    double pnl;
} Trade;
```

Encodes invariants:

- Exactly 3 fields
    
- Known types
    
- Known layout
    
- Known ownership
    
- Known lifetime
    

The compiler _enforces_ these.

A pointer tuple:

```c
void *t[3];
```

Encodes **nothing**.

All meaning is in your head.

---

## This is why “higher layer” exists

Higher layers:

- Do not prove memory safety
    
- Do not parse bytes
    
- Do not own allocation
    
- Do not enforce bounds
    

They **consume invariants** established below.

That’s not weakness — it’s architecture.

---

## A sentence that sums it up

> **Assumptions are informal.  
> Invariants are contractual.**

NGINX runs on invariants, not hope.

---

## Bringing it home to your work

Your HTTP parser:

- Establishes invariants → pointer math OK
    

Your trading logic / request handling:

- Consumes invariants → structs win
    

If you mixed them:

- You’d either re-check everything (slow)
    
- Or rely on assumptions (unsafe)
    

NGINX refuses to do either.

---

If you want, next we can:

- Annotate your code and explicitly label invariants
    
- Identify which functions _must_ prove them
    
- Decide exactly where pointer arithmetic is justified in _your_ codebase