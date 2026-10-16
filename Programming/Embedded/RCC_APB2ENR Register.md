Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_apb2enr_register

On the **STM32F446RE**, **RCC_APB2ENR** stands for:

> **Reset and Clock Control – APB2 Peripheral Clock Enable Register**

---

## What It Does

It **enables or disables the clock signal** to peripherals connected to the **APB2 bus**.

If a peripheral’s bit in RCC_APB2ENR is:

- `0` → Clock disabled → Peripheral inactive
    
- `1` → Clock enabled → Peripheral operational
    

No clock → the peripheral’s internal logic does not run.

---

## Where APB2 Fits in the Clock Tree

`HSI / HSE     ↓    PLL     ↓   SYSCLK     ↓    AHB     ↓   APB2 (high-speed peripheral bus)     ↓  ADC, SPI1, USART1, TIM1, etc.`

RCC_APB2ENR controls that final gating stage.

---

## Typical APB2 Peripherals (STM32F446)

Common peripherals on APB2 include:

- TIM1 (advanced-control timer)
    
- USART1
    
- SPI1
    
- ADC1 / ADC2 / ADC3
    
- SYSCFG
    
- EXTI (via SYSCFG)
    

Each one has a corresponding enable bit in RCC_APB2ENR.

---

## Example: Enable USART1

`RCC->APB2ENR |= (1 << 4);   // Enable USART1 clock`

Or with HAL:

`__HAL_RCC_USART1_CLK_ENABLE();`

If you forget this:

- Baud rate registers won’t take effect
    
- Transmit won’t work
    
- Peripheral appears “dead”
    

---

## ⚡ Why APB2 Is Separate From APB1

STM32 splits peripherals by performance level:

| Bus  | Typical Speed | Type of Peripherals                  |
| ---- | ------------- | ------------------------------------ |
| APB1 | Lower speed   | General timers, I2C, basic UART      |
| APB2 | Higher speed  | ADC, high-speed SPI, advanced timers |

For STM32F446:

- APB1 max ≈ 45 MHz
    
- APB2 max ≈ 90 MHz
    

Higher-performance peripherals need higher clock ceilings.

---

## What’s Actually Happening

When you set a bit in RCC_APB2ENR:

- The clock gate feeding that peripheral opens
    
- The internal state machines receive clock pulses
    
- Register writes begin to take effect
    
- Power consumption increases slightly
    

When cleared:

- Clock pulses stop
    
- Peripheral freezes in its current state
    

---

## Mental Rule in Embedded Systems

Before touching any peripheral:

1. Enable its clock (AHB / APB1 / APB2)
    
2. Configure registers
    
3. Enable peripheral internally
    
4. Enable interrupts if needed
    

Clock enable is always step one.

---

## Important Pattern You’re Seeing

You now have:

- RCC_AHB1ENR → Enables GPIO, DMA, CRC
    
- RCC_APB1ENR → Enables low-speed peripherals
    
- RCC_APB2ENR → Enables high-speed peripherals
    

This is the MCU’s **clock gating architecture**.

Everything is layered and controlled.