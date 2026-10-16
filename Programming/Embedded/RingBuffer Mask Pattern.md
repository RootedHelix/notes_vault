Related to [[Embedded]], [[Embedded Ring Buffer]], 
#embedded #embedded_ring_buffer #embedded_pattern #ringbuffer_mask_pattern

The ring buffer uses this pattern to write: 

```C
#define MASK (HLX_RING_BUFFER_SIZE - 1)

bool hlx_ringbuf_write(hlx_ringbuf_t* rb, const uint8_t* data, uint32_t len) {  
    for (uint32_t i = 0; i < len; i++) {  
        uint32_t next_head = (rb->head + 1) & MASK;  
  
        if (next_head == rb->tail) {  
            return false;  
        }  
        rb->buf[rb->head] = data[i];  
        rb->head = next_head;  
    }    return true;  
}
```

- We use the `uint32_t next_head = (rb->head + 1) & MASK;` in this instead of using the modulo operator like this: `next_head = (rb->head + 1) % SIZE;`
		- We are attempting to get wrap around indexing 


The goal is to implement **wrap-around indexing**.

Normally you would write:

next_head = (rb->head + 1) % SIZE;

But `%` (modulus) is **slow on many CPUs**.

Instead we use:

(index) & (SIZE - 1)

> This **only works when SIZE is a power of two**.

Example with size = 8:

SIZE = 8  
MASK = 7  
MASK = 0b111

---

### Without wrap

head = 3  
  
head + 1 = 4  
4 & 7 = 4

---

#### When wrap happens

head = 7  
  
head + 1 = 8  
  
8 = 0b1000  
MASK = 0b0111 

1000 & 0111 = 0000  


---


Result:

next_head = 0

So the index **wraps automatically**.

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