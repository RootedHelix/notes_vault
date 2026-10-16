Related to [[Embedded]], [[Embedded Logging]], [[Embedded Ring Buffer]], [[HAL Library]]
#embedded #embedded_logging #hal_uart_txcpltcallback #hal_library 

`TxCpltCallback` stands for:

```
Transmit Complete Callback
```

Let's break it down.

---

# 1. The Name Components

### `Tx`

Short for **Transmit**.

In UART terminology:

```
TX → transmit (send data out)
RX → receive (read incoming data)
```

---

### `Cplt`

Short for **Complete**.

STM32 HAL uses abbreviations in many callback names:

|Abbreviation|Meaning|
|---|---|
|Cplt|Complete|
|Rx|Receive|
|Tx|Transmit|
|Err|Error|

---

### `Callback`

A **callback function** is a function that **the HAL library calls automatically** when an event occurs.

So the HAL says:

```
"When transmit finishes, call this function."
```

---

# 2. Full Meaning

```c
HAL_UART_TxCpltCallback()
```

Means:

```
"Call this function when a UART transmission is complete."
```

---

# 3. When It Runs

For DMA transmission:

```c
HAL_UART_Transmit_DMA(...)
```

Flow:

```
CPU starts DMA
↓
DMA hardware sends bytes to UART
↓
DMA finishes
↓
interrupt fires
↓
HAL calls HAL_UART_TxCpltCallback()
```

So the callback runs **after the entire DMA transfer has finished**.

---

# 4. In Your Logger

Your code uses it perfectly:

```
DMA finishes
↓
HAL_UART_TxCpltCallback()
↓
consume bytes from ring buffer
↓
start next DMA transfer
```

Which creates a **continuous streaming logger**.

---

# 5. Why HAL Uses This Design

The HAL uses callbacks because it is **event-driven**.

Instead of:

```
while(UART not finished)
    wait
```

it does:

```
start DMA
return immediately
later → callback runs
```

This is **non-blocking I/O**, which is critical for embedded systems.

---

# 6. Related UART Callbacks

You’ll often see these:

|Function|Meaning|
|---|---|
|`HAL_UART_TxCpltCallback`|transmit finished|
|`HAL_UART_RxCpltCallback`|receive finished|
|`HAL_UART_ErrorCallback`|UART error occurred|
|`HAL_UARTEx_RxEventCallback`|receive event (DMA idle mode)|

---

# 7. Small Historical Note

These shortened names (`Tx`, `Rx`, `Cplt`) come from **hardware driver naming conventions** going back decades.

You'll see the same pattern in:

- networking stacks
    
- OS kernels
    
- hardware drivers.
    

---

If you'd like, I can also show you **what actually happens inside the HAL and DMA interrupt when `TxCpltCallback` is triggered**. It's a really nice example of **hardware → interrupt → driver → user callback flow**.