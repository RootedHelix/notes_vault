Related to [[Embedded]], [[Coniguring USART2 TX DMA  in the ioc Using CubeMX]], [[HAL_UART_Transmit_DMA]], [[USART2]]

#embedded #usart2 


# AI Response

Yes — that is the correct place. What you are seeing is the **DMA request mapping table** for USART2 TX.

Once you pressed **Add → USART2_TX**, CubeMX creates a DMA channel/stream entry. Now you just need to verify the parameters in those columns.

### The fields you should configure

You should now see something like:

|Request|Stream|Direction|Priority|
|---|---|---|---|
|USART2_TX|(DMA stream/channel)|Memory to Peripheral|Low/Medium/High|

#### 1. **Stream**

This corresponds to the **DMA channel/stream that services the USART2 TX request**.

CubeMX usually auto-selects the correct one depending on the MCU.

Examples:

- On **STM32F4** → something like `DMA1 Stream 6`
    
- On **STM32F1/F0/L0** → something like `DMA1 Channel 7`
    

If CubeMX already filled this automatically, **leave it**.

#### 2. **Direction**

For UART transmit via DMA it **must be**

```
Memory to Peripheral
```

Reason:  
DMA copies bytes from your RAM buffer → UART data register.

#### 3. **Priority**

This determines arbitration if multiple DMA transfers occur simultaneously.

Typical choice:

```
Low
```

You could choose **Medium** if:

- You have multiple DMA users
    
- You want UART logging to preempt others
    

For most firmware projects **Low is fine**.

### What should happen next

After setting that row:

1. **Save the `.ioc` file**
    
2. CubeMX regenerates code
    

Then verify two important things were generated.

#### In `dma.c`

You should see initialization of the DMA stream/channel.

Example pattern:

```c
hdma_usart2_tx.Instance = DMA1_Stream6;
hdma_usart2_tx.Init.Direction = DMA_MEMORY_TO_PERIPH;
```

#### In `usart.c`

CubeMX links the DMA handle to the UART handle:

```c
__HAL_LINKDMA(&huart2, hdmatx, hdma_usart2_tx);
```

This linkage is **critical**. Without it, `HAL_UART_Transmit_DMA()` will fail.

### One more important thing to check

CubeMX sometimes requires enabling the **DMA interrupt**.

Check:

```
System Core → NVIC
```

Make sure the interrupt corresponding to your DMA stream/channel is enabled, e.g.:

```
DMA1 Stream6 global interrupt
```

### Final sanity test in your code

You should be using:

```c
HAL_UART_Transmit_DMA(&huart2, buffer, length);
```

If you instead call:

```c
HAL_UART_Transmit(...)
```

then DMA will **never be used**, even if everything is configured.

---

If you want, show me **one screenshot of the DMA row** you just created (especially the **Stream column**) and I can confirm whether CubeMX picked the correct DMA stream for your MCU.