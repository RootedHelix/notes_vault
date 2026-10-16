Related to [[Embedded]], [[HAL Library]], [[Embedded Logging]], [[HAL_UART_Transmit]]
#embedded #hal_library #hal_uart_transmit


# DMA _ Pattern

[[DMA_Pattern]]

### HAL_UART_Transmit_DMA()

means:

> **Transmit data over UART using Direct Memory Access instead of the CPU.**

# HAL_UART_Transmit vs HAL_UART_Transmit_DMA

### `HAL_UART_Transmit()`

- **blocking**
- simple
- good for quick tests / tiny messages / startup debug

### `HAL_UART_Transmit_DMA()`

- **non-blocking**
- more efficient
- better for production-style logging or continuous output
- requires a bit more setup and care

**DMA** stands for **Direct Memory Access**.


---

# The Problem Without DMA

Normally when sending data over UART, the CPU has to **feed each byte** to the UART peripheral.

Example conceptually:

```
CPU -> UART -> serial line
```

Pseudo code:

```c
for (i = 0; i < len; i++) {
    while (!(UART->SR & TX_READY));
    UART->DR = buffer[i];
}
```

So the CPU:

1. Waits
    
2. Writes byte
    
3. Waits
    
4. Writes byte
    
5. Waits again
    

This wastes CPU time.

---

# What DMA Does

DMA is **hardware that moves memory for you**.

Instead of the CPU doing the transfer:

```
CPU -> UART
```

You configure DMA once:

```
Memory -> DMA controller -> UART peripheral
```

Now the hardware automatically moves the data.

```
RAM --------\
             -> DMA -> UART -> serial line
UART -------/
```

The CPU can continue doing other work.

---

# What `HAL_UART_Transmit_DMA()` Does

When you call:

```c
HAL_UART_Transmit_DMA(&huart2, buffer, len);
```

the HAL configures the DMA controller to:

```
Source:      buffer in RAM
Destination: UART transmit register
Length:      len bytes
```

Then the DMA engine performs:

```
buffer[0] -> UART
buffer[1] -> UART
buffer[2] -> UART
...
```

automatically.

The CPU **returns immediately**.

---

# Timeline Comparison

### Blocking transmit

```
CPU: send byte
CPU: wait
CPU: send byte
CPU: wait
CPU: send byte
CPU: wait
```

CPU is busy the entire time.

---

### DMA transmit

```
CPU: configure DMA
CPU: continue program

DMA: send byte
DMA: send byte
DMA: send byte
```

CPU is free.

---

# Real Hardware Picture (STM32)

Inside STM32 you have:

```
CPU core
RAM
DMA controller
UART peripheral
```

DMA acts like a **hardware memory copy engine**.

```
RAM -> DMA -> UART_TX register
```

---

# Why DMA Is Powerful

DMA can transfer data between:

```
Memory -> UART
Memory -> SPI
Memory -> ADC
ADC -> Memory
SPI -> Memory
Memory -> Memory
```

So you can do things like:

```
ADC sampling continuously
DMA storing samples in RAM
CPU processes data later
```

This is extremely common in embedded systems.

---

# Example UART Logging With DMA

Professional logging might look like:

```
Application logs -> ring buffer
Ring buffer -> DMA -> UART
```

DMA sends data in the background.

---

# Real Embedded Systems Use DMA Everywhere

DMA is commonly used for:

### UART

```
debug logging
```

### SPI

```
display drivers
flash memory
```

### ADC

```
sensor sampling
```

### I2S

```
audio streaming
```

### Ethernet

```
network packet buffers
```

---

# Key Insight

DMA is essentially:

> **A tiny hardware processor dedicated to copying memory.**

It can move data **without waking the CPU**.

That’s why DMA is one of the **most important performance features in microcontrollers**.

---

If you'd like, I can also explain something **very useful for embedded engineers**:

**Why ring buffers + DMA together are the gold standard for high-performance embedded logging and data streaming.**