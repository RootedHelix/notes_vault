Related to [[Embedded]] , [[System Clock Tree]], 
#embedded #ahb

## AHB 

**AHB** stands for **Advanced High-performance Bus**.

It’s part of the **ARM AMBA bus architecture**, and it’s the **main high-speed internal data highway** inside many microcontrollers.

In simple terms:

> 🛣️ AHB is the fast internal bus that connects the CPU to high-speed peripherals and memory.

---

## Where It Comes From

AHB is defined by ARM as part of the AMBA specification.

It’s used in Cortex-M microcontrollers like the STM32F446RE (which you’re using).

---

## What AHB Connects

Typically connected to AHB:

- Flash memory
    
- SRAM
    
- DMA
    
- GPIO
    
- High-speed peripheral
		Such as : 
		`- GPIOA`
		`- GPIOB`
		`- GPIOC`
		`- GPIOD`
		`- GPIOE`
		`- DMA1`
		`- DMA2`
		`- CRC`
		`- Backup SRAM`
    

Each has one bit in RCC_AHB1ENR.
    
- Sometimes external memory controllers
    

---

## Visual Concept

![Image](https://community.st.com/t5/image/serverpage/image-id/122181i39714574C135C159/image-size/large/is-moderation-mode/true?px=999&v=v2)

![Image](https://www.allaboutcircuits.com/uploads/articles/Introduction_to_AMBA_fig01.png)

![Image](https://www.researchgate.net/publication/269304013/figure/fig1/AS%3A668933396918297%401536497558893/Advanced-Microcontroller-Bus-Architecture-1.png)

![Image](https://www.allaboutcircuits.com/uploads/articles/Introduction_to_AMBA_fig02.png)

---

## 🧩 How It Fits in STM32 Clock Tree

In STM32 (like F4 series), you typically have:

```
SYSCLK
   ↓
AHB Prescaler
   ↓
HCLK (AHB clock)
   ↓
APB1 / APB2 prescalers
```

So:

- **SYSCLK** comes from PLL
    
- It gets divided by an AHB prescaler
    
- That result is called **HCLK**
    
- HCLK clocks:
    
    - CPU
        
    - Memory interface
        
    - AHB peripherals
        

---

## ⚡ Example

If:

- SYSCLK = 84 MHz
    
- AHB prescaler = 1
    

Then:

```
HCLK = 84 MHz
```

If prescaler = 2:

```
HCLK = 42 MHz
```

Now the CPU and memory run at 42 MHz.

---

## Why AHB Matters (Important for You)

Since you're moving toward bare-metal thinking:

AHB affects:

- CPU speed
    
- Memory access timing
    
- DMA throughput
    
- GPIO toggle speed
    
- Interrupt latency
    

If AHB is slow → everything slows down.

---

## AHB vs APB

|Bus|Purpose|Speed|
|---|---|---|
|AHB|High-speed core bus|Fast|
|APB1|Low-speed peripherals|Slower|
|APB2|Higher-speed peripherals|Medium|

Typical usage:

- AHB → SRAM, DMA
    
- APB1 → UART2, I2C
    
- APB2 → SPI1, ADC
    

---

## Real-World Impact Example

If your timer is on APB1 and APB1 is divided by 2:

Your timer frequency changes.

And if APB prescaler > 1:

STM32 timers may automatically multiply clock by 2 internally.

This is one of those "why is my timer twice as fast?" moments.

---

## Big Picture

Inside your MCU there is no “magic.”

There are just:

- Clock sources
    
- PLL
    
- Prescalers
    
- Buses (AHB/APB)
    
- Peripherals
    

AHB is simply:

> The main high-speed internal backbone connecting CPU to memory and major hardware blocks.

---