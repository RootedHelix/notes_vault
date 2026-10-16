Related to [[Embedded]], [[DMA Concurrency issues]], [[DMA_zero_copy_Pattern]] [[Embedded Ring Buffer]]
#embedded #dma_concurrency_issues #hal_uart_transmit_dma #embedded_ring_buffer #__get_primask

`PRIMASK` is a **CPU register on ARM Cortex-M** that controls whether **normal maskable interrupts** are allowed to run.

In your code, it is being used to make the ring-buffer write into a **tiny critical section** without accidentally changing the interrupt state incorrectly.

Here is the relevant part again:

```c
uint32_t primask = __get_PRIMASK();
__disable_irq();

bool ok = hlx_ringbuf_write(&log_rb, (const uint8_t*)tmp, write_len);

if (primask == 0U) {
    __enable_irq();
}
```

## What `PRIMASK` is

Think of `PRIMASK` as a **1-bit interrupt mask state** for the processor.

It is basically:

- `0` → interrupts are enabled
    
- `1` → interrupts are disabled for normal maskable interrupts
    

So it answers the question:

> “Right now, before I do anything, is the CPU accepting interrupts or not?”

---

## What `__get_PRIMASK()` does

`__get_PRIMASK()` reads that current CPU register value and returns it to your code.

So this line:

```c
uint32_t primask = __get_PRIMASK();
```

means:

> “Save the current interrupt mask state before I change anything.”

That way, after your critical section, you know what the old state was.

---

## Why that matters

Because your function may be called in different contexts.

### Case 1: normal code path

Suppose interrupts are currently enabled.

Then:

```c
__get_PRIMASK()  --> 0
```

You disable interrupts, do the write, and afterward you should turn them back on.

That is why this works:

```c
if (primask == 0U) {
    __enable_irq();
}
```

Since `primask` was `0`, you re-enable them.

---

### Case 2: interrupts were already disabled before you entered

Suppose some higher-level code already disabled interrupts before calling your function.

Then:

```c
__get_PRIMASK()  --> 1
```

Now your function disables interrupts again, but they were already off.

After the critical section, you **must not** call `__enable_irq()`, because that would undo the caller’s protection.

So:

```c
if (primask == 0U) {
    __enable_irq();
}
```

does nothing in that case.

That means:

> “Only re-enable interrupts if I was the one who turned them off.”

That is the whole point.

---

## What `__disable_irq()` is doing with this

This line:

```c
__disable_irq();
```

sets the interrupt mask so normal interrupts cannot preempt your code.

So the sequence is:

1. Read old interrupt state with `__get_PRIMASK()`
    
2. Disable interrupts
    
3. Perform shared-state update safely
    
4. Restore only if the old state said interrupts were originally on
    

---

## Why not just blindly enable afterward

Because this would be wrong:

```c
__disable_irq();
hlx_ringbuf_write(...);
__enable_irq();
```

That code assumes interrupts were enabled before the function started.

But if they were already disabled, then calling `__enable_irq()` would incorrectly turn them back on.

That can break other critical sections and create subtle bugs.

---

## The mental model

A good way to think about it is:

- `__get_PRIMASK()` = “what was the door state before I touched it?”
    
- `__disable_irq()` = “close the door”
    
- critical section happens
    
- conditional `__enable_irq()` = “reopen the door only if I was the one who closed it”
    

---

## In your specific logger

In your logger, this means:

- before touching the shared ring buffer, you save whether interrupts were already enabled
    
- you then temporarily prevent interrupt-based preemption
    
- you safely update `log_rb`
    
- you restore the system to the same interrupt state it had before entering the function
    

So `PRIMASK` is not protecting the buffer by itself.

It is giving you a way to **safely save and restore interrupt state** around the protected operation.

## Tiny summary

`PRIMASK` is the Cortex-M register that tells whether normal interrupts are masked.

`__get_PRIMASK()` reads that value so your code can remember the previous interrupt state.

In your function, it is used so that after disabling interrupts for the ring-buffer write, you only re-enable them if they were enabled before the critical section started.

If you want, I can also explain the difference between `PRIMASK`, `BASEPRI`, and `FAULTMASK`, because that is a very useful Cortex-M distinction.
