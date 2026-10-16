Related to [[Embedded]], [[HAL Library]], [[Embedded Logging]], 
#embedded #hal_library #hal_uart_transmit

# Function Prototype

From the STM32 HAL library:

```c
HAL_StatusTypeDef HAL_UART_Transmit(
    UART_HandleTypeDef *huart,
    uint8_t *pData,
    uint16_t Size,
    uint32_t Timeout
);
```

---

# What Each Parameter Means

### 1️`UART_HandleTypeDef *huart`

Pointer to the UART peripheral configuration.

Example:

```c
extern UART_HandleTypeDef huart2;
```

This struct contains things like:

- baud rate
    
- registers
    
- hardware instance
    
- configuration state
    

Example call:

```c
HAL_UART_Transmit(&huart2, data, len, 100);
```

---

### 2️`uint8_t *pData`

Pointer to the **buffer of bytes you want to send**.

Example:

```c
uint8_t msg[] = "Hello\n";
```

---

### 3️ `uint16_t Size`

Number of bytes to send.

Example:

```c
HAL_UART_Transmit(&huart2, msg, 6, 100);
```

---

### 4 `uint32_t Timeout`

Maximum time (ms) the function will **wait for transmission to finish**.

Example:

```
100 = wait up to 100ms
HAL_MAX_DELAY = wait forever
```

---

# What Happens Internally

The HAL function basically does something like this:

```
for each byte:
    wait until UART TX register empty
    write byte to UART data register
```

Conceptually:

```c
for (i = 0; i < Size; i++) {
    while (!TX_READY)
        ;  // wait

    UART_DR = pData[i];
}
```

Then it waits for the **final transmission complete flag**.

---

# Why It Is Blocking

Because it **waits for hardware flags** before continuing.

During this time:

```
CPU is stuck inside HAL_UART_Transmit()
```

Example timeline:

```
CPU
 |
 | HAL_UART_Transmit()
 |    wait...
 |    wait...
 |    wait...
 v
return
```

If you send **50 bytes**, your CPU might sit there for **several milliseconds**.

---

# Why Embedded Engineers Avoid It for Logging

If you log like this:

```c
HAL_UART_Transmit(&huart2, msg, len, 100);
```

Every log call **pauses the CPU**.

That’s why we instead do:

```
log() -> ring buffer
ring buffer -> background UART transmit
```

So the main system **never blocks**.

---

# Non-Blocking Versions

STM32 HAL also provides **better options**.

### Interrupt version

```
HAL_UART_Transmit_IT()
```

Transmission occurs via **interrupts**.

---

### DMA version (best)

```
HAL_UART_Transmit_DMA()
```

Transmission occurs via **DMA hardware**.

```
CPU -> DMA -> UART
```

CPU becomes free immediately.

---

# Professional Embedded Logging Approach

Usually one of these:

### Simple system

```
ring buffer -> HAL_UART_Transmit()
```

(but done in a low-priority task)

---

### Better system

```
ring buffer -> HAL_UART_Transmit_IT()
```

---

### Best system
[[HAL_UART_Transmit_DMA]]

```
ring buffer -> HAL_UART_Transmit_DMA()
```

Very common in professional firmware.

---

# One More Important Insight

The HAL is just a **software wrapper around registers**.

The real hardware transmit register is:

```
USARTx->TDR
```

Low-level drivers sometimes skip HAL entirely:

```c
USART2->TDR = byte;
```

Because it’s **faster and deterministic**.

---

If you'd like, I can also show you something **very cool and relevant to your projects**:

**How professional embedded loggers can transmit UART without interrupts, DMA, or blocking — using only a ring buffer and a tiny state machine.**

It’s a pattern used in **bootloaders and kernels**, and it’s very elegant.