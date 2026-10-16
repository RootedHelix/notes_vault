Related to [[Embedded]], [[HAL_UART_Transmit_DMA]]
#embedded #hal_library #hal_uart_transmit_dma #embedded_pattern

What you’re referring to is a **zero-copy DMA ring buffer pattern**. Instead of copying data from the ring buffer into `log_dma_buf`, the DMA reads **directly from the ring buffer memory**.

> Find `log_dma_buf` in the embedded projects for the stm32 code - logging.c

This is common in:

- UART drivers
    
- network stacks
    
- high-performance loggers
    
- kernels (Linux uses similar buffer techniques)
    

The key idea is: **DMA transmits the contiguous region already sitting in the ring buffer.**

---

# 1. Current Design (Copying Version)

Right now your system looks like this:

```
printf
   ↓
ring buffer
   ↓ (peek + copy)
log_dma_buf
   ↓
DMA → UART
```

That means every transmission performs:

```
memcpy
```

Which is unnecessary if DMA can read directly from the buffer.

---

# 2. Zero-Copy Architecture

Instead you do this:

```
printf
   ↓
ring buffer
   ↓
DMA reads directly from rb->buf
```

So the flow becomes:

```
write → ring buffer
flush → start DMA at rb->buf[tail]
DMA complete → advance tail
```

No intermediate buffer.

---

# 3. Important Constraint

DMA can only read **contiguous memory**.

But a ring buffer may wrap:

```
[......DATA......]
        tail
               head
```

So we only send the **first contiguous chunk**.

Example:

```
buffer size = 1024

tail = 900
head = 100
```

Data exists in two regions:

```
900 → 1023
0   → 100
```

DMA must transmit them **in two separate transfers**.

---

# 4. New Helper Function

Instead of `peek()`, we return a pointer to the first chunk.

```c
uint32_t hlx_ringbuf_linear_read(hlx_ringbuf_t *rb, uint8_t **ptr)
{
    if (rb->head >= rb->tail) {

        *ptr = &rb->buf[rb->tail];
        return rb->head - rb->tail;

    } else {

        *ptr = &rb->buf[rb->tail];
        return HLX_RING_BUFFER_SIZE - rb->tail;
    }
}
```

This gives:

```
pointer to contiguous memory
+
length
```

---

# 5. DMA Flush Function (Zero-Copy)

```c
void hlx_log_flush_dma(void)
{
    if (log_tx_busy) {
        return;
    }

    uint8_t *ptr;
    uint32_t n = hlx_ringbuf_linear_read(&log_rb, &ptr);

    if (n == 0) {
        return;
    }

    if (HAL_UART_Transmit_DMA(&huart2, ptr, n) == HAL_OK) {

        log_tx_busy = true;
        log_dma_len = n;
    }
}
```

Notice:

```
ptr points directly into ring buffer
```

No copy.

---

# 6. DMA Completion (unchanged)

```c
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
    if (huart == &huart2) {

        hlx_ringbuf_consume(&log_rb, log_dma_len);

        log_tx_busy = false;

        hlx_log_flush_dma();
    }
}
```

---

# 7. Why This Is Better

### Removes a memory copy

Old:

```
ring buffer → memcpy → dma buffer → UART
```

New:

```
ring buffer → UART
```

---

### Reduces CPU load

No copy loop.

---

### Lower latency

DMA starts immediately.

---

### Higher throughput

Especially when logs are large.

---

# 8. Visual Model

```
RING BUFFER MEMORY

+-----------------------------------+
|   data being transmitted by DMA   |
+-----------------------------------+
            ↑
           tail

DMA reads directly from here
```

When DMA completes:

```
tail += transmitted bytes
```

---

# 9. Why Professional Drivers Use This

Most high-performance systems avoid copies whenever possible.

Examples:

- Linux kernel network stack
    
- NGINX send buffers
    
- embedded RTOS logging systems
    
- high-speed UART drivers
    

The principle is called:

```
zero-copy I/O
```

---

# 10. When Not to Use Zero-Copy

Your current design with `log_dma_buf` is actually **perfectly acceptable** if:

- logs are small
    
- simplicity matters more than peak performance.
    

Zero-copy is just the **next optimization step**.

---

✅ **My honest recommendation**

Your current implementation is already **very good engineering**.  
If this were production firmware, the zero-copy version would simply be a **performance upgrade**, not a necessity.

---

If you'd like, I can also show you something very interesting:

**Why your `MASK` trick makes the ring buffer extremely fast compared to `%` modulo buffers**, and why almost every high-performance ring buffer in systems programming uses that exact trick.