Related to [[DMA]], [[Embedded]], [[__disable_irq()]], [[__get_Primask()]]
#embedded #hal_uart_transmit_dma #__get_primask


# AI Response
ISR means **Interrupt Service Routine**.

In your current logger context, it refers to code that runs in response to a hardware interrupt instead of normal foreground execution.

For your example, this callback:

```c
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart == &huart2) {
        hlx_ringbuf_consume(&log_rb, log_dma_len);
        log_dma_len = 0U;
        log_tx_busy = false;
        hlx_log_flush_dma();
    }
}
```

is effectively part of the **ISR path**.

More precisely:

- the UART/DMA hardware finishes transmitting
    
- a hardware interrupt fires
    
- the HAL interrupt handler runs
    
- the HAL eventually calls `HAL_UART_TxCpltCallback(...)`
    
- your code runs there in interrupt context
    

So even though you did not write the low-level vector handler yourself, your callback is still running as part of interrupt handling.

## Why that matters

Code running in ISR context is different from normal code:

- it can interrupt foreground code at almost any point
    
- it should usually run quickly
    
- it shares data with non-ISR code carefully
    
- it often requires `volatile`, critical sections, or other synchronization discipline
    

In your logger:

- `hlx_log_write()` is normal code / foreground code
    
- `HAL_UART_TxCpltCallback()` is ISR-context code
    
- both touch shared state like:
    
    - `log_tx_busy`
        
    - `log_dma_len`
        
    - `log_rb`
        

That is why I kept referring to ISR safety.

## Mental model

Think of it like this:

- **foreground code** = normal program flow
    
- **ISR code** = urgent hardware-driven side path that can preempt the foreground
    

In embedded systems, a huge amount of design work is really about coordinating those two worlds correctly.

## In your project specifically

When I say:

> protect shared state between foreground and ISR

I mean protect access between code like:

```c
hlx_log_write();
hlx_log_flush_dma();
```

and code like:

```c
HAL_UART_TxCpltCallback();
```

because they can interleave in ways that produce races if not designed carefully.

So in this context:

**ISR = the interrupt-driven callback path associated with DMA/UART completion.**