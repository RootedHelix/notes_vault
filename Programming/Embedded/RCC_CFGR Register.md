Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_cfgr_register

On the **STM32F446RE**, **RCC_CFGR** means:

> **Reset and Clock Control – Clock Configuration Register**

If ENR registers **turn clocks on**,  
CFGR decides **how clocks are routed and divided**.

---

## What It Does (Big Picture)

RCC_CFGR controls:

1. **System clock source selection**
    
2. **AHB prescaler**
    
3. **APB1 prescaler**
    
4. **APB2 prescaler**
    
5. **Peripheral clock routing (some cases)**
    
6. **Microcontroller Clock Output (MCO)**
    

Think of it as the **traffic controller** of the clock tree.

---

##  Where It Sits in the Clock Architecture

`HSI / HSE     ↓    PLL     ↓   (selected in CFGR)     ↓   SYSCLK     ↓   AHB prescaler (CFGR)     ↓   APB1 prescaler (CFGR)   APB2 prescaler (CFGR)`

CFGR controls the switching and the divisions.

---

## 1️ Selecting System Clock (SW bits)

These bits choose what becomes **SYSCLK**:

- 00 → HSI
    
- 01 → HSE
    
- 10 → PLL
    

Example:

`RCC->CFGR |= (2 << 0);  // Select PLL as system clock`

You must also wait until the hardware confirms the switch (SWS bits).

---

##  2️ Prescalers (Clock Dividers)

CFGR sets how much the clocks are divided.

Example fields:

- HPRE → AHB prescaler
    
- PPRE1 → APB1 prescaler
    
- PPRE2 → APB2 prescaler
    

Example:

`RCC->CFGR |= (5 << 10);  // Divide APB1 by 4`

This reduces peripheral bus frequency.

---

## ⚡ Why Prescalers Matter

Example:

- SYSCLK = 84 MHz
    
- APB1 max allowed = 45 MHz
    

So you divide:

84 MHz / 2 = 42 MHz → safe for APB1

Without CFGR prescalers, you would violate hardware limits.

---

##  What Makes CFGR Important

ENR registers say:

> “Power this peripheral.”

CFGR says:

> “At what speed does the entire system run?”

If you misconfigure CFGR:

- MCU may run too slow
    
- Or too fast
    
- Or violate bus limits
    
- Or fail to switch clock sources
    

---

## Practical Example (Typical Setup)

For STM32F446 at 84 MHz:

1. Enable HSE
    
2. Configure PLL
    
3. Set APB1 prescaler to /2
    
4. Set APB2 prescaler to /1
    
5. Switch SYSCLK to PLL via CFGR
    

CFGR handles steps 3–5.

---

## How to Think About RCC_CFGR

It’s the **central configuration hub** for:

- Clock routing
    
- Bus scaling
    
- System frequency behavior
    

If CR enables oscillators,  
and ENR enables peripherals,

CFGR configures the **clock distribution network**.