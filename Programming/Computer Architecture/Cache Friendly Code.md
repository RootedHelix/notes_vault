Cache-friendly code is code that **plays nicely with the CPU caches** so that most memory accesses hit the fast caches instead of slow main RAM.

Put another way:

> **Cache-friendly code maximizes locality** (temporal + spatial) so the CPU can reuse data it has already pulled into cache.

---

## Quick refresher: what’s “cache”?

Between CPU and RAM you have small, fast memories:

- L1 cache: tiny, insanely fast
    
- L2, L3: bigger, slower than L1, still way faster than RAM
    
- RAM: huge, but **hundreds of cycles** away
    

Data moves in **cache lines** (commonly 64 bytes). If you touch one `int` that lives in that line, the CPU pulls the whole 64 bytes in. If you then touch neighbors in memory, you get them “for free” (from cache).

So:

- **Spatial locality** = use data that’s near other data you just used.
    
- **Temporal locality** = reuse the same data again soon.
    

Cache-friendly code leans hard on both.

---

## What cache-friendly code usually looks like

### 1. Walk arrays in order (stride-1 access)

```c
// Cache-friendly
for (size_t i = 0; i < n; i++) {
    sum += a[i];
}
```

This is great because:

- `a[0], a[1], a[2], ...` are contiguous.
    
- Each cache line brings in multiple elements.
    
- Next iteration hits data that’s already in cache.
    

Compare to:

```c
// Cache-unfriendly (large stride)
for (size_t i = 0; i < n; i++) {
    sum += a[i * 16]; // skip around in memory
}
```

If `a` is large, you may only use **one element per cache line** and then throw it away. Lots of misses.

---

### 2. Use data layouts that match your access pattern

**Array of structs (AoS):**

```c
typedef struct {
    float x, y, z;
    float mass;
} Body;

Body bodies[N];

for (size_t i = 0; i < N; i++) {
    // good if you always need *all* fields of each Body
    bodies[i].x += vx[i];
    bodies[i].y += vy[i];
}
```

If you always use all fields of each `Body`, AoS is fine.

**Struct of arrays (SoA):**

```c
typedef struct {
    float *x, *y, *z;
    float *mass;
} Bodies;

Bodies b;

// good if your hot loop only needs x and y:
for (size_t i = 0; i < N; i++) {
    b.x[i] += vx[i];
    b.y[i] += vy[i];
}
```

Now `x` and `y` live in tight, separate contiguous arrays. Cache loves this, SIMD loves this.

Rule of thumb:

- If you usually operate on **one object at a time**, AoS can be OK.
    
- If you usually operate on **one field across many objects**, SoA is usually more cache-friendly.
    

---

### 3. Avoid random pointer chasing in hot paths

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;

// Linked list traversal: often cache-unfriendly
int sum_list(Node *head) {
    int sum = 0;
    for (Node *p = head; p != NULL; p = p->next) {
        sum += p->value;  // p->next can be anywhere in memory
    }
    return sum;
}
```

Each `next` might jump to a totally unrelated memory location → many cache misses.

If possible, store the data in a **contiguous array** and iterate by index instead. Linked structures are fine when you truly need them, but they’re often bad for cache.

---

### 4. Loop ordering matters (2D arrays)

Assume row-major layout like C:

```c
int a[ROWS][COLS];

// Cache-friendly: row-major access
for (int i = 0; i < ROWS; i++) {
    for (int j = 0; j < COLS; j++) {
        sum += a[i][j];
    }
}
```

This walks memory in the order it’s laid out.

Bad version:

```c
// Cache-unfriendly: column-major style on row-major data
for (int j = 0; j < COLS; j++) {
    for (int i = 0; i < ROWS; i++) {
        sum += a[i][j];   // jumps by COLS each time
    }
}
```

Now you’re skipping around in memory; each access may hit a different cache line.

---

### 5. Blocking / tiling for large data

When working with big matrices, you often **tile** operations so a working block fits into cache.

Very rough idea:

```c
// multiply C = A * B, but in tiles
for (int ii = 0; ii < N; ii += BLOCK) {
  for (int jj = 0; jj < N; jj += BLOCK) {
    for (int kk = 0; kk < N; kk += BLOCK) {
      // work on a BLOCK×BLOCK submatrix
      for (int i = ii; i < ii + BLOCK; i++) {
        for (int j = jj; j < jj + BLOCK; j++) {
          for (int k = kk; k < kk + BLOCK; k++) {
            C[i][j] += A[i][k] * B[k][j];
          }
        }
      }
    }
  }
}
```

You keep a small chunk of A, B, and C hot in cache, instead of streaming over huge matrices in cache-unfriendly patterns.

---

### 6. For multithreading: avoid false sharing

Even if two threads touch **different variables**, if those variables sit on the **same cache line**, the cache system will bounce that line between cores.

```c
typedef struct {
    int counter;
} ThreadData;

ThreadData data[NUM_THREADS];

// If each thread increments data[i].counter in a tight loop,
// they might all be on the same cache line → false sharing.
```

Cache-friendly multi-threaded code often:

- Pads per-thread data so each thread’s hot variables are on **different cache lines**.
    
- Example: align structs to 64 bytes and add padding.
    

---

## How to think about it when writing C

When you write code, ask:

1. **Am I touching memory in a predictable, sequential way?**
    
2. **Does my data layout match how I actually use it?**
    
3. **Am I using arrays (contiguous) where I can, instead of random pointer structures?**
    
4. **In nested loops, does my inner loop walk contiguous memory?**
    
5. **For multi-threading, are threads hammering the same cache lines?**
    

If the answer to those is “yes / mostly”, your code is likely cache-friendly.

---

If you want, next we can take a real snippet from your Lapideus code (or a toy function) and I can point out where it’s cache-friendly and where you could reshape the data/loops for better cache behavior.