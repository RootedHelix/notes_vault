Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_ahb1enr_register

On the **STM32F446RE** (and other STM32F4 parts), **RCC_AHB1ENR** stands for:

> **Reset and Clock Control – AHB1 Peripheral Clock Enable Register**

---

## 🧠 What It Does (Conceptually)

It **turns ON or OFF the clock signal** to peripherals connected to the **AHB1 bus**.

If the clock to a peripheral is disabled:

- Its registers won’t respond
    
- Reads may return 0
    
- Writes won’t take effect
    
- The peripheral is effectively “dead”
    

Enabling the clock powers its internal logic.

---

## Where It Fits in the Clock Tree

`Clock Source (HSI/HSE)      
`↓`        
`PLL`         
`↓`       
`SYSCLK`         
`↓`        
`AHB`
`↓`
`AHB1 Peripheral`

RCC_AHB1ENR controls that last step.

---

# 📌 What Peripherals Are on AHB1?

On STM32F446, AHB1 commonly includes:

- GPIOA
    
- GPIOB
    
- GPIOC
    
- GPIOD
    
- GPIOE
    
- DMA1
    
- DMA2
    
- CRC
    
- Backup SRAM
    

Each has one bit in RCC_AHB1ENR.

---

## Example: Enabling GPIOA

To use GPIOA:

`RCC->AHB1ENR |= (1 << 0);   // Enable GPIOA clock`

Bit 0 corresponds to GPIOA.

Equivalent HAL macro:

`__HAL_RCC_GPIOA_CLK_ENABLE();`

If you forget this step:

Your GPIO configuration will not work.

This is one of the most common beginner mistakes.

---

## What’s Actually Happening Electrically?

The microcontroller gates the clock signal.

When disabled:

- No clock toggles inside that peripheral
    
- No state machines advance
    
- Power consumption drops
    

Clock gating is a power-saving and control mechanism.

---

## Why This Matters Architecturally

Embedded systems are layered:

- You configure a peripheral
    
- But only after enabling its clock
    
- Which only works after the bus clock is configured
    
- Which only works after system clock is configured
    

This enforces dependency thinking.