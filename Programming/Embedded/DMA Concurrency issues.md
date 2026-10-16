Related to [[STM32 Week1]], [[Embedded]], [[HAL_UART_Transmit_DMA]]
#embedded #stm32 #hal_library #hal_uart_transmit_dma #dma_concurrency_issues

Currently my architecture does not protect against writes from multiple places throughout the system. The suggested fixes consist of protecting writes with a critical section: 

## Critical section for writes 

Absolutely — here is the full text cleaned up and reformatted so it’s easier to read, then I’ll expand on how it fixes the concurrency issue.

---

A good critical section for the writes would be **small** and wrapped only around the **ring-buffer enqueue**, not around `vsnprintf()`.

## The idea

You want to protect this kind of operation:

```c
hlx_ringbuf_write(&log_rb, data, len);
```

from being interrupted by another writer.

On STM32/Cortex-M, the usual simple approach is:

1. Save the current interrupt enable state
    
2. Disable interrupts
    
3. Do the shared-state update
    
4. Restore the previous interrupt state
    

---

## What that looks like

In your logger write function, the pattern is typically:

```c
#include "cmsis_gcc.h"
```

or just rely on the CMSIS functions already available through the STM32 headers.

Then:

```c
#include <hlx_logging.h>
#include <hlx_ringbuffer.h>
#include <stdio.h>
#include <stdarg.h>
#include "cmsis_gcc.h"

#define HLX_LOG_BUF_SIZE    128
#define HLX_LOG_CHUNK_SIZE  64

static hlx_ringbuf_t log_rb;
static volatile bool log_tx_busy = false;
static volatile uint32_t log_dma_len = 0U;

void hlx_log_init(void) {
    hlx_ringbuf_init(&log_rb);
    log_tx_busy = false;
    log_dma_len = 0U;
}

void hlx_log_write(int level, const char* fmt, ...) {
    (void)level;

    char tmp[HLX_LOG_BUF_SIZE];

    va_list args;
    va_start(args, fmt);
    int len = vsnprintf(tmp, sizeof(tmp), fmt, args);
    va_end(args);

    if (len > 0) {
        uint32_t write_len = (len >= (int)sizeof(tmp)) ? (sizeof(tmp) - 1U) : (uint32_t)len;

        uint32_t primask = __get_PRIMASK();
        __disable_irq();

        (void)hlx_ringbuf_write(&log_rb, (const uint8_t*)tmp, write_len);

        if (primask == 0U) {
            __enable_irq();
        }
    }
}
```

---

## What `PRIMASK` is doing

`__get_PRIMASK()` tells you whether interrupts were already disabled.

- `0` → interrupts were enabled
    
- `1` → interrupts were disabled
    

That matters because you don’t want to blindly re-enable interrupts if you entered this code from a context that had already disabled them for another reason.

So this pattern:

```c
uint32_t primask = __get_PRIMASK();
__disable_irq();

/* critical section */

if (primask == 0U) {
    __enable_irq();
}
```

means:

> Restore interrupt state only if I was the one who disabled them.

That’s the polite embedded thing to do.

---

## Why not just do this?

```c
__disable_irq();
hlx_ringbuf_write(...);
__enable_irq();
```

Because if interrupts were already disabled before entering the function, you’d accidentally turn them back on.

That can create some very nasty bugs.

---

## Very important: keep the critical section short

This is good:

```c
vsnprintf(...);   // outside critical section

__disable_irq();
hlx_ringbuf_write(...);
__enable_irq();
```

This is bad:

```c
__disable_irq();
vsnprintf(...);
hlx_ringbuf_write(...);
__enable_irq();
```

Because formatting can take relatively long, and you don’t want interrupts off that long.

---

## An even cleaner helper approach

If you want to avoid repeating this pattern, you can make tiny helpers.

For example:

```c
static uint32_t hlx_log_enter_critical(void) {
    uint32_t primask = __get_PRIMASK();
    __disable_irq();
    return primask;
}

static void hlx_log_exit_critical(uint32_t primask) {
    if (primask == 0U) {
        __enable_irq();
    }
}
```

Then use them like:

```c
uint32_t primask = hlx_log_enter_critical();
(void)hlx_ringbuf_write(&log_rb, (const uint8_t*)tmp, write_len);
hlx_log_exit_critical(primask);
```

That’s often nicer to read.

---

## Should you also protect `hlx_log_flush_dma()`?

Maybe, but not necessarily first.

The **highest-value protection** is around the write path, because that prevents multiple producers from corrupting `head`.

For your current design, that’s the main thing to fix.

If later you want to make the logger more robust, you can also protect small parts of:

- `log_tx_busy`
    
- `log_dma_len`
    
- DMA start decision
    

But I would start with the write critical section only.

---

## One subtle point

A critical section around `hlx_ringbuf_write()` prevents **concurrent writers** from stepping on each other.

It does **not** automatically solve all possible race conditions in the whole logger, but it handles the most common and important one for your current setup.

---

## What about putting the critical section inside `hlx_ringbuf_write()`?

You _could_, but I generally would **not** do that unless this ring buffer is only ever used in interrupt-shared situations.

Why?

Because a ring buffer is a generic data structure. It’s usually better if the caller decides whether protection is needed.

That keeps the ring buffer reusable.

So I’d prefer:

- `hlx_ringbuf_write()` stays generic
    
- `hlx_log_write()` adds protection because logging needs it
    

---

## One more practical improvement

Right now you ignore the return value:

```c
(void)hlx_ringbuf_write(...)
```

That’s okay temporarily, but eventually you may want:

- a dropped-log counter
    
- or at least detection of overflow
    

Because if the queue is full, logs will get dropped or partially written.

---

## Bottom line

A solid write critical section looks like this:

```c
uint32_t primask = __get_PRIMASK();
__disable_irq();

hlx_ringbuf_write(&log_rb, data, len);

if (primask == 0U) {
    __enable_irq();
}
```

And you should place it **only around the enqueue step**, not around the formatting work.

If you want, I can show you the **next best improvement**: making `hlx_ringbuf_write()` all-or-nothing so a full buffer doesn’t leave you with half a log line.

---

# Now let’s expand on how this fixes the concurrency issue

The key issue is that your logger has **shared mutable state**.

For the ring buffer, that is mainly:

- `rb->head`
    
- `rb->tail`
    
- `rb->buf[...]`
    

The dangerous part is that `hlx_ringbuf_write()` is **not atomic**. It does several steps:

```c
uint32_t next_head = (rb->head + 1) & MASK;

if (next_head == rb->tail) {
    return false;
}

rb->buf[rb->head] = data[i];
rb->head = next_head;
```

That sequence is safe only if one writer runs at a time.

---

# What goes wrong without protection

Imagine this:

- Main loop calls `hlx_log_write("hello")`
    
- Halfway through enqueueing, a UART ISR or timer ISR also calls `hlx_log_write("error")`
    

Now you have **two writers** both reading and modifying `rb->head`.

Example:

### Initial state

```c
head = 10
tail = 4
```

### Writer A starts

It computes:

```c
next_head = 11
```

Before it stores the byte and updates `head`, an interrupt happens.

### Writer B starts

It also sees:

```c
head = 10
tail = 4
next_head = 11
```

So it also thinks slot 10 is free.

Then both writers can:

- write into the same slot
    
- update `head` inconsistently
    
- interleave bytes from two different messages
    

That causes:

- corrupted log text
    
- dropped bytes
    
- impossible-to-debug behavior
    
- ring buffer state corruption
    

This is the concurrency problem.

---

# What the critical section changes

When you do:

```c
uint32_t primask = __get_PRIMASK();
__disable_irq();

hlx_ringbuf_write(&log_rb, data, len);

if (primask == 0U) {
    __enable_irq();
}
```

you make the enqueue operation effectively **non-interruptible**.

That means:

- Writer A enters
    
- interrupts are disabled
    
- Writer B cannot preempt if B is ISR-based
    
- Writer A finishes updating the ring buffer
    
- interrupts are restored
    
- only then can Writer B run
    

So instead of:

```c
A: read head
B: read head
A: write byte
B: write byte
A: update head
B: update head
```

you get:

```c
A: complete whole enqueue
B: complete whole enqueue afterward
```

That turns the operation into a serialized update.

---

# Why this helps specifically on Cortex-M

On a Cortex-M microcontroller, most concurrency issues in firmware come from:

- main context vs interrupt context
    
- interrupt vs interrupt
    
- DMA completion callback vs foreground code
    

Disabling interrupts is a very common way to protect a tiny shared-data update because:

- it is simple
    
- it is fast
    
- it avoids needing an RTOS mutex
    
- it is appropriate for short critical sections
    

So for your logger, this is exactly the right first-level fix.

---

# What it does not fix

This is important.

This fix protects against **simultaneous writers**, but it does **not** solve every logger race automatically.

For example, you still have other shared logger state:

```c
static volatile bool log_tx_busy = false;
static volatile uint32_t log_dma_len = 0U;
```

And functions like:

- `hlx_log_flush_dma()`
    
- `hlx_HAL_UART_TxCpltCallback()`
    

also share state.

So the write critical section solves:

- producer/producer corruption of the ring buffer
    

But it does not fully solve:

- producer/consumer interaction bugs
    
- DMA state race bugs
    
- flush/start callback timing issues
    

Still, it fixes the **most important structural bug first**.

---

# Why this is the right first fix

Because ring buffer corruption is catastrophic.

If the queue bookkeeping gets corrupted, you can get:

- broken logs forever after
    
- phantom full/empty states
    
- bytes overwritten silently
    

Whereas some DMA-start race conditions may only cause:

- a missed transmission start
    
- an extra flush call
    
- temporary delay
    

So protecting the ring-buffer writer gives you the biggest stability improvement first.

---

# Important nuance: this assumes interrupt-driven concurrency

This works if your concurrency model is mainly:

- foreground code
    
- ISRs
    

That is probably true for your STM32 logger.

If later you move to:

- RTOS tasks
    
- multicore system
    
- more advanced lock-free producer model
    

then disabling interrupts alone may not be enough.

But for STM32 bare metal or light HAL-based firmware, it is the standard answer.

---

# The deeper reason this works

Your ring buffer write is a **read-modify-write sequence** on shared state.

Those are exactly the operations that need protection.

You are not protecting the whole function because the whole function is not dangerous.

This part is safe:

```c
vsnprintf(tmp, sizeof(tmp), fmt, args);
```

because `tmp` is a local stack buffer.

This part is dangerous:

```c
hlx_ringbuf_write(&log_rb, (const uint8_t*)tmp, write_len);
```

because `log_rb` is shared global state.

That distinction is the real embedded systems lesson here:

- local state usually does not need locking
    
- shared mutable state does
    

---

# Very concrete before/after

## Before

Two contexts can enter `hlx_log_write()` at once and both modify `log_rb`.

Result:

- corrupted head movement
    
- overwritten bytes
    
- mixed log lines
    

## After

Only one context at a time can perform the enqueue.

Result:

- ring buffer state remains coherent
    
- each writer sees the updated head from the previous writer
    
- logs may still be dropped if full, but not corrupted by concurrent writes
    

That is a huge improvement.

---

# One more subtle benefit

Even if only one byte write is “atomic” on the CPU, the **whole logical enqueue operation is not atomic**.

So this critical section makes the entire _logical transaction_ atomic at the firmware level.

That is the important part.

You’re not protecting a single instruction.  
You’re protecting a **state transition**.

That is a much better way to think about concurrency in embedded systems.

---

# The best mental model

Think of the ring buffer as a tiny shared database.

A write operation is:

1. Check if there is room
    
2. Write data
    
3. Advance head
    

That whole thing must happen as one uninterrupted transaction.

Your critical section is what enforces that transaction boundary.

---

# Final takeaway

Yes — this fixes the concurrency issue in the sense that it prevents **multiple writers from corrupting the ring buffer**.

More precisely, it fixes the most important concurrency bug:

- **non-atomic shared-state update of `head` and buffer contents**
    

It does that by serializing writers through a very short interrupt-masked critical section.

The cleanest summary is:

- `vsnprintf()` stays outside
    
- ring-buffer enqueue goes inside
    
- the critical section is tiny
    
- shared state stays coherent
    
- your logger becomes much more reliable
    

The next step after this would be to tighten up the **DMA side state machine**, because that is the other place where concurrency can still bite you.
