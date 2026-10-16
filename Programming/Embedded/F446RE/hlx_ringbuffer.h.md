Related to [[Embedded]], [[Embedded Ring Buffer]], [[Hollix]]

#embedded #embedded_ring_buffer #hollix 

When defining `HLX_RING_BUFFER_SIZE`, a value of **512 bytes** was selected. This size is commonly used for DMA-backed logging buffers and, as a **power of two**, enables efficient index wrapping via bitmasking instead of modulo operations.

The ring buffer is defined as:

struct hlx_ringbuf_s {  
    uint8_t buf[HLX_RING_BUFFER_SIZE];  
    volatile uint32_t head;  
    volatile uint32_t tail;  
};

- The buffer is implemented as a `uint8_t` array to provide **byte-level granularity**, which is appropriate for UART/DMA transmission and avoids alignment constraints associated with wider types.
    
- The `head` and `tail` indices are marked `volatile` because they may be **modified by different execution contexts** (e.g., main loop and ISR or DMA completion callback). This prevents the compiler from caching their values in registers and ensures each access reflects the most recent state in memory


---
Matched .c = [[hlx_ringbuffer.c]] 