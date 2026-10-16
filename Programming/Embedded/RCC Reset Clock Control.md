Related to [[Embedded]],[[HSE High Speed External Oscillator or Crystal]] [[HSI High Speed Internal Oscillator]], [[AHB Advanced High Performance Bus]], [[HSE High Speed External Oscillator or Crystal]], [[APB Advanced Peripheral Bus]], [[Prescalar]]  
#embedded #rcc_reset_clock_control

## What is RCC?

**RCC = Reset and Clock Control**

On your STM32F446RE, RCC is the hardware peripheral that:

1. Controls all clock sources (HSI, HSE, PLL)
    
2. Selects which clock feeds the CPU
    
3. Sets bus prescalers (AHB, APB1, APB2)
    
4. Enables/disables clocks to peripherals
    
5. Handles some reset logic
    

Think of RCC as the **power + timing control center** of the chip.

---

## Why RCC Exists

Microcontrollers don’t just “run at a speed.”

They have:

- Internal oscillator (HSI)
    
- External crystal (HSE)
    
- PLL multipliers
    
- Different buses (AHB, APB1, APB2)
    
- Dozens of peripherals
    

All of those need clocks.

And they must be:

- Enabled
    
- Selected
    
- Divided
    
- Multiplied
    
- Routed
    

That’s what RCC does.

---

## Core Registers

[[RCC_CR Register]]  
[[RCC_PLLCFGR Register]]
[[RCC_CFGR Register]]
[[RCC_AHB1ENR Register]]        
[[RCC_APB1ENR Register]]        
[[RCC_APB2ENR Register]]

RCC registers divide responsibility:

| Register    | Responsibility               |
| ----------- | ---------------------------- |
| RCC_CR      | Control oscillator power     |
| RCC_PLLCFGR | Configure PLL multiplication |
| RCC_CFGR    | Select and divide clocks     |
| RCC_*ENR    | Enable peripheral clocks     |

CR = Turn engines on  
CFGR = Route and scale power  
ENR = Deliver power to peripherals



## Two Major Roles of RCC

### 1️⃣ System Clock Configuration

RCC controls:

```
Clock Source → PLL → SYSCLK → AHB → APB → Peripheral
```

Example:

- HSE = 8 MHz
    
- PLL multiplies to 84 MHz
    
- SYSCLK = 84 MHz
    
- APB1 = 42 MHz
    
- APB2 = 84 MHz
    

That entire chain is configured through RCC registers.

---

### 2️ Peripheral Clock Enabling

Even if you configure GPIO properly…

It will not work until you enable its clock.

Example (bare metal):

```c
RCC->AHB1ENR |= (1 << 0);  // Enable GPIOA clock
```

If you don’t enable it:

- The peripheral appears dead.
    
- Writes to registers do nothing.
    

This is a classic beginner bug.

---

## Why Clock Enabling Exists

Power saving.

Peripherals consume dynamic power when clocked.

So STM32 defaults most peripherals to **clock disabled**.

You explicitly turn them on through RCC.

That’s why RCC is foundational.

---

## Where RCC Lives

It’s memory mapped like all peripherals.

For STM32F4:

- Base address: `0x40023800`
    
- Registers like:
    
    - RCC_CR
        
    - RCC_PLLCFGR
        
    - RCC_CFGR
        
    - RCC_AHB1ENR
        
    - RCC_APB1ENR
        
    - RCC_APB2ENR
        

CubeMX configures these for you.

Bare-metal engineers eventually write to them directly.

---

## Mental Model

Think of RCC as:

- The clock router
    
-  The peripheral power switchboard
    
- ⏱ The timing authority of the MCU
    

Without RCC:

- CPU doesn’t know how fast to run
    
- Peripherals don’t receive clocks
    
- Timers don’t tick
    
- UART can’t generate baud rate
    

---

## Why This Matters For You

When you:

- Configure timers
    
- Configure UART
    
- Configure ADC
    
- Switch clock sources
    
- Enter low power modes
    

You are interacting with RCC — directly or indirectly.

---

## Simple Analogy

Imagine a factory:

- HSE = power plant
    
- PLL = voltage transformer
    
- SYSCLK = main power line
    
- AHB/APB = branch circuits
    
- RCC = the breaker panel
    

You don’t plug machines directly into the power plant.

You route through the panel.

---        
