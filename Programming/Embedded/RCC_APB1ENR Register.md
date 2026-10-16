Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_apb1enr_register

On the **STM32F446RE**, **RCC_APB1ENR** means:

> **Reset and Clock Control – APB1 Peripheral Clock Enable Register**

---

# 🧠 What It Does

It **enables or disables the clock** to peripherals connected to the **APB1 bus**.

If a peripheral’s bit in RCC_APB1ENR is:

- `0` → Clock disabled → Peripheral does nothing
    
- `1` → Clock enabled → Peripheral becomes operational
    

No clock = no register activity.

---

# 🏗 Where APB1 Sits in the Clock Tree

`HSI / HSE     ↓    PLL     ↓   SYSCLK     ↓    AHB     ↓  APB1 (low-speed peripheral bus)     ↓  Timers, UARTs, I2C, etc.`

RCC_APB1ENR gates clocks at that last step.

---

## Typical APB1 Peripherals (STM32F446)

Common examples:

- TIM2, TIM3, TIM4, TIM5
    
- USART2, USART3
    
- I2C1, I2C2, I2C3
    
- SPI2, SPI3
    
- PWR (Power control)
    
- DAC
    

Each one has a corresponding enable bit.

---

## Example: Enable TIM2

`RCC->APB1ENR |= (1 << 0);  // Enable TIM2 clock`

Or using HAL:

`__HAL_RCC_TIM2_CLK_ENABLE();`

If you skip this step:

- Timer registers won’t increment
    
- Interrupts won’t fire
    
- Peripheral appears broken
    

---

## ⚡ Why APB1 Is Separate From AHB1

STM32 divides buses by performance:

|Bus|Purpose|
|---|---|
|AHB|High-speed core + GPIO + DMA|
|APB1|Lower-speed peripherals|
|APB2|Higher-speed peripherals|

APB1 usually runs at a lower maximum frequency than APB2.

For STM32F446:

- APB1 max = 45 MHz
    
- APB2 max = 90 MHz
    

This saves power and balances performance.

---

## 🔬 What’s Happening Physically?

The MCU uses **clock gating**:

When a peripheral’s enable bit = 0:

- Internal clock signal is stopped
    
- Internal state machines freeze
    
- Power usage decreases
    

When enabled:

- Clock toggles resume
    
- Peripheral logic becomes active
    

---

## Mental Model

Before configuring any peripheral:

1. Enable its clock (RCC_APB1ENR)
    
2. Configure registers
    
3. Enable peripheral
    
4. Optionally enable interrupts
    

Clock enable is always step 1.