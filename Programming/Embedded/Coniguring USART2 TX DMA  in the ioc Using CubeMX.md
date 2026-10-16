Related to [[Embedded]] , [[ioc File]], [[HAL_UART_Transmit_DMA]]
#usart2 #embedded 

Yes. In CubeMX, you want to verify that **USART2 has a TX DMA request explicitly attached**, not just that USART2 itself is enabled.

Use this sequence:

1. **Open your `.ioc` file** in STM32CubeMX / CubeIDE.
    
2. Go to **Pinout & Configuration**.
    
3. In the left panel, expand **Connectivity** and click **USART2**.
    
4. Confirm USART2 is enabled in an appropriate mode, usually:
    
    - **Asynchronous**
        
    - **TX** or **TX/RX**, depending on your project
        

Then check the DMA binding:

5. While still in the **USART2 configuration** window, find the **DMA Settings** tab or section.
    
6. Click **Add** under DMA requests. 
    [[DMA Request Mapping Table for USART2 CubeMX]]
    
7. In the new DMA row, make sure the request is specifically:
    
    - **USART2_TX**
        

If you do **not** see a row for `USART2_TX`, then TX DMA is **not configured yet**.

What that row should generally indicate:

- Direction should be **Memory to Peripheral**
    
- Mode is usually **Normal** for one-shot transmissions
    
- Peripheral increment should be **disabled**
    
- Memory increment should usually be **enabled**
    
- Data width is usually **Byte / Byte** for UART text or byte streams
    

After that, check interrupts:

8. Go to **System Core → NVIC**
    
9. Make sure the relevant **DMA channel/stream interrupt  ---- or named DMA1 stream6 global interupt** is enabled if your generated HAL flow expects interrupt-driven DMA completion. ST’s DMA setup flow also calls out enabling the DMA interrupt when using DMA with interrupts. ([STMicroelectronics](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_DMA?utm_source=chatgpt.com "Getting started with DMA - stm32mcu - ST wiki"))
    

Then regenerate and verify code generation:

10. Save the `.ioc` file so Cube regenerates code.
    
11. Open:
    

- `usart.c`
    
- `dma.c`
    

What you want to see in generated code:

- A DMA handle for the USART2 TX path, such as something like:
    
    - `hdma_usart2_tx`
        
- A link between UART and DMA, typically via:
    
    - `__HAL_LINKDMA(&huart2, hdmatx, hdma_usart2_tx);`
        

If that linkage is missing, CubeMX did not actually wire TX DMA into USART2.

Final runtime check:  
12. In your code, make sure you are calling:

```c
HAL_UART_Transmit_DMA(&huart2, buffer, length);
```

and not:

```c
HAL_UART_Transmit(&huart2, buffer, length, timeout);
```

Because even if CubeMX configures DMA correctly, **DMA will not be used unless you call the DMA transmit API**. ST’s UART getting-started material distinguishes normal blocking transmit, interrupt transmit, and DMA-based transfer as separate paths. ([STMicroelectronics](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_UART?utm_source=chatgpt.com "Getting started with UART - stm32mcu - ST wiki"))

A fast sanity check:

- If `DMA Settings` for USART2 has **no `USART2_TX` row**, it is not configured.
    
- If generated code has **no `hdma_usart2_tx`** and no `__HAL_LINKDMA(...hdmatx...)`, it is not configured.
    
- If your application never calls `HAL_UART_Transmit_DMA()`, DMA TX is not being used.
    

If you want, paste:

- a screenshot of your CubeMX **USART2 DMA Settings**
    
- and the generated `MX_USART2_UART_Init()` / `MX_DMA_Init()`
    

and I’ll verify the configuration line by line.

# Verification 
### After generating code, verify three things

1. **dma.c**
    

You should see something similar to:

`hdma_usart2_tx.Instance = DMA1_Stream6;`
`hdma_usart2_tx.Init.Direction = DMA_MEMORY_TO_PERIPH;`

> These were under : **`/home/helix/Projects/embedded_collection/stm32/F446RE_Test/Core/Src/stm32f4xx_hal_msp.c`** 


2. **usart.c**
    

Look for the DMA linkage:

__HAL_LINKDMA(&huart2, hdmatx, hdma_usart2_tx);

> I found a macro for this under **`/home/helix/Projects/embedded_collection/stm32/F446RE_Test/Drivers/STM32F4xx_HAL_Driver/Inc/stm32f4xx_hal_def.h`**

3. **NVIC configuration**
    

In `dma.c` you should also see something like:

HAL_NVIC_SetPriority(DMA1_Stream6_IRQn, 0, 0);  
HAL_NVIC_EnableIRQ(DMA1_Stream6_IRQn);

> I Found these under **`/home/helix/Projects/embedded_collection/stm32/F446RE_Test/Drivers/STM32F4xx_HAL_Driver/Src/stm32f4xx_hal_cortex.c`**

That confirms CubeMX enabled the DMA interrupt.

>And then inside : `/home/helix/Projects/embedded_collection/stm32/F446RE_Test/Core/Src/stm32f4xx_it.c`

I have: 

```C
void DMA1_Stream6_IRQHandler(void)  
{  
  /* USER CODE BEGIN DMA1_Stream6_IRQn 0 */  
  
  /* USER CODE END DMA1_Stream6_IRQn 0 */  HAL_DMA_IRQHandler(&hdma_usart2_tx);  
  /* USER CODE BEGIN DMA1_Stream6_IRQn 1 */  
  
  /* USER CODE END DMA1_Stream6_IRQn 1 */}
```


### Good habit when working with CubeMX

Any time you change:

- pins
    
- peripherals
    
- DMA
    
- clocks
    
- interrupts
    

you should **save the `.ioc` and regenerate code**.

---

If you'd like, the next useful step is verifying that **USART2 DMA actually transmits**, which takes about **30 seconds** and immediately confirms the configuration is correct.