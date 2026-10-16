Related to [[Embedded]], [[hlx_ringbuffer.h]], [[Embedded Ring Buffer]]
#embedded #hlx_ringbuffer #embedded_ring_buffer 


---

# HLX Ring Buffer (SPSC, DMA-Compatible)

## Overview

This module implements a **fixed-size circular buffer** optimized for embedded systems and DMA-driven I/O.

- **Model:** Single Producer / Single Consumer (SPSC)

		- This is the transport layer for the the logging layer above this. 
			- The logging layer is the producer and the DMA / UART Driver is the
			     consumer

    
- **Indexing:** Power-of-two size with bitmask wrapping
    
- **Concurrency:** Safe across main context and ISR/DMA callback _without locks_, assuming correct role separation
    
- **Primary use case:** UART/DMA logging or streaming pipelines
    

---

## Compile-Time Requirements

```
#define HLX_RING_BUFFER_SIZE 512
#define MASK (HLX_RING_BUFFER_SIZE - 1)
```

### Constraints

- `HLX_RING_BUFFER_SIZE` **must be a power of two**
    
- `MASK` enables wrap-around using:
    
    ```c
    index = (index + 1) & MASK;
    ```
    
    This avoids costly modulo operations.
    

---

## Data Structure

```c
struct hlx_ringbuf_s {
    uint8_t buf[HLX_RING_BUFFER_SIZE];
    volatile uint32_t head;
    volatile uint32_t tail;
};
```

### Semantics

- `head`: write index (producer-owned)
    
- `tail`: read index (consumer-owned)
    
- Buffer is considered:
    
    - **Empty:** `head == tail`
        
    - **Full:** `(head - tail) & MASK == MASK`
        

---

## Initialization

### `hlx_ringbuf_init`

```c
void hlx_ringbuf_init(hlx_ringbuf_t* rb);
```

Initializes the ring buffer.

#### Behavior

- Sets `head = 0`
    
- Sets `tail = 0`
    

#### Notes

- Must be called before any read/write operations
    

---

## Write Path (Producer)

### `hlx_ringbuf_write`

```c
bool hlx_ringbuf_write(hlx_ringbuf_t* rb, const uint8_t* data, uint32_t len);
```

Writes `len` bytes into the buffer.

#### Returns

- `true` → write successful
    
- `false` → insufficient space (no partial writes)
    

#### Behavior

```c
used = (head - tail) & MASK
free = MASK - used
```

- Ensures capacity before writing
    
- Writes byte-by-byte
    
- Advances `head` with masking
    

#### Constraints

- Producer must be the **only writer of `head`**
    
- No internal synchronization (lock-free assumption)
    

---

## Read Path (Consumer)

### `hlx_ringbuf_read`

```c
uint32_t hlx_ringbuf_read(hlx_ringbuf_t* rb, uint8_t* dest, uint32_t max_len);
```

Reads up to `max_len` bytes into `dest`.

#### Returns

- Number of bytes actually read
    

#### Behavior

- Stops when:
    
    - buffer becomes empty, or
        
    - `max_len` reached
        
- Advances `tail`
    

---

### `hlx_ringbuf_is_empty`

```c
bool hlx_ringbuf_is_empty(hlx_ringbuf_t* rb);
```

#### Returns

- `true` if buffer contains no data
    

---

## DMA-Oriented API

These functions support **zero-copy or staged DMA transfers**.

---

### `hlx_ringbuf_peek`

```c
uint32_t hlx_ringbuf_peek(hlx_ringbuf_t* rb, uint8_t* out, uint32_t max_len);
```

Copies data **without advancing `tail`**.

#### Use Case

- Preparing a DMA buffer before committing consumption
    

#### Behavior

- Reads from a local snapshot of `tail`
    
- Does not modify buffer state
    

---

### `hlx_ringbuf_consume`

```c
void hlx_ringbuf_consume(hlx_ringbuf_t* rb, uint32_t len);
```

Advances `tail` by `len`.

#### Behavior

- Caps `len` at buffer size
    
- Performs masked increment
    

#### Use Case

- Called after DMA transfer completes
    

---

### `hlx_ringbuf_linear_read`

```c
uint32_t hlx_ringbuf_linear_read(hlx_ringbuf_t* rb, uint8_t** ptr);
```

Returns a **contiguous readable region** starting at `tail`.
#### Outputs

Always
> `*ptr` → direct pointer to the contiguous data,  >>where the readable data starts<<

Conditional
- `0U` : If the ringbuffer is empty `rb->tail = rb->head`
- `The exact length of the memory in the ringbuffer `: if the ring buffer head/index is larger than the ring buffer tail `rb->head > rb->tail`
- `By Default:` `End of the Ring buffer to the tail Size`: Tell the caller how many bytes are available at the returned pointer before wrapping is required.  
	`return RING_BUFFER_SIZE - rb->tail;` 

#### Returns

- Length of contiguous region
    

#### Behavior

- If `head > tail`:
    
    - contiguous region = `head - tail`
        
- If wrapped:
    
    - contiguous region = `HLX_RING_BUFFER_SIZE - tail`
        

#### Use Case

- Direct DMA transmission without copying
    

---

## Concurrency Model

This implementation assumes:

- **Single Producer**
    
    - Writes data
        
    - Modifies `head`
        
- **Single Consumer**
    
    - Reads/consumes data
        
    - Modifies `tail`
        

### Typical Mapping

|Context|Role|
|---|---|
|Main loop|Producer|
|DMA ISR|Consumer|

or vice versa.

---

## Memory Model Notes

- `head` and `tail` are marked `volatile` to ensure:
    
    - No register caching across contexts
        
    - Each access reflects memory state
        

### Important

- `volatile` **does not guarantee atomicity or memory ordering**
    
- This implementation relies on:
    
    - Naturally atomic 32-bit accesses (on Cortex-M)
        
    - Strict SPSC discipline
        

---

## Edge Cases / Limitations

- No overwrite mode (write fails if insufficient space)
    
- No multi-producer or multi-consumer support
    
- No explicit memory barriers (may be required on more complex systems)
    
- Byte-wise writes may be suboptimal for large transfers (can be optimized)
    

---

## Summary

This ring buffer is:

- Efficient (masking instead of modulo)
    
- Lock-free under SPSC assumptions
    
- DMA-friendly (linear read + consume model)
    
- Suitable for embedded logging and streaming pipelines
    

---

If you want to take this one step further, the next upgrade would be:

- **Interrupt-safe batching (disable IRQ around head update)**
    
- **Bulk write optimization (memcpy with wrap handling)**
    
- **Memory barrier annotations for portability**
    

That would move this from “good embedded implementation” to **production-grade firmware infrastructure**.
