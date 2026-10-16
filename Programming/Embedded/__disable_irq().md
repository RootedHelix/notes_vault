Related to [[Ensuring Race Safe Logger]], [[Embedded Logging]], [[Embedded]], [[HAL_UART_Transmit_DMA]], [[__get_Primask()]]

#diable_irq #embedded #race_conditions #dma_concurrency_issues 


# Personal Notes

- irq stands for interrupt request
- Manipulates the PRIMASK register on the cortex-M 





# AI Response
## `__disable_irq()` disables interrupts globally on the CPU

On Cortex-M, it sets the **PRIMASK** bit so that **normal maskable interrupts stop being serviced**.

So after:

```

__disable_irq();
```

your code keeps running, but interrupt handlers generally **won’t interrupt it** until interrupts are re-enabled.

---

## CRITICAL
#embedded_pattern 
Usually to create a tiny **critical section** around shared data:

```C
uint32_t primask = __get_PRIMASK();  
__disable_irq();  
  
/* touch shared state safely */  
  
if (primask == 0U) {  
    __enable_irq();  
}
```

That means:

- save whether interrupts were already disabled
- disable them
- do short protected work
- only re-enable if they were previously enabled

That pattern avoids accidentally turning interrupts back on when some outer code had already disabled them.

---

## In plain English

It means:

> “For the next few instructions, don’t let an interrupt handler jump in and modify shared state under me.”

---

## Important nuance

It does **not**:

- stop DMA hardware
- stop the CPU
- stop faults like HardFault/NMI
- magically make code thread-safe in every possible environment

It mainly prevents **normal interrupt preemption** on that core.

---

## For your logger context

If your callback can run from an interrupt and your main code also touches the ring buffer, then:

```

__disable_irq();
```

is a simple way to say:

> “Don’t let the UART/DMA completion interrupt fire in the middle of this queue update.”

That’s why it’s common in small bare-metal embedded code.

---

## Caution

You should keep interrupts disabled for **very short** sections only.

Bad idea:

```

__disable_irq();/* do formatting, blocking UART, long loop, heavy work */__enable_irq();
```

Good idea:

```

__disable_irq();/* update a couple shared variables */__enable_irq();
```

Because while interrupts are disabled:

- timers are delayed
- UART/DMA callbacks are delayed
- latency-sensitive code suffers

---

## Short version

`__disable_irq()` means:

> **“Temporarily block normal interrupts so this code can safely touch shared state.”**

If you want, I can also explain:

- `__get_PRIMASK()`
- `__enable_irq()`
- why disabling IRQs is different from using `volatile`

