Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_cr_register

On the **STM32F446RE**, **RCC_CR** stands for:

> **Reset and Clock Control – Clock Control Register**

If **RCC_CFGR** decides _which clock to use_,  
then **RCC_CR** controls the **raw clock sources themselves**.

---

## What RCC_CR Does (High Level)

It controls:

1. Turning oscillators ON/OFF
    
2. Checking if oscillators are ready
    
3. Enabling the PLL
    
4. Checking PLL lock status
    

It is the **power switch panel for clock generators**.

---

## Where It Fits in the Clock Tree

`[ HSI ]   [ HSE ]      ↓        ↓          [ PLL ]              ↓          (selected later in CFGR)              ↓            SYSCLK`

RCC_CR controls the blocks at the top: HSI, HSE, PLL.

---

## Key Bits Inside RCC_CR

### HSI (Internal 16 MHz Oscillator)

- **HSION** → Turn HSI ON
    
- **HSIRDY** → HSI Ready flag
    

After reset:

- HSI is ON by default
    
- SYSCLK = HSI
    

---

###  HSE (External Crystal)

- **HSEON** → Turn HSE ON
    
- **HSERDY** → HSE Ready flag
    

Example:

`RCC->CR |= (1 << 16);   // Enable HSE while (!(RCC->CR & (1 << 17)));  // Wait for HSE ready`

You must wait for RDY before using it.

---

### PLL (Phase Locked Loop)

- **PLLON** → Turn PLL ON
    
- **PLLRDY** → PLL locked flag
    

Example:

`RCC->CR |= (1 << 24);   // Enable PLL while (!(RCC->CR & (1 << 25)));  // Wait for PLL lock`

PLL must lock before switching SYSCLK to it.

---

## ⚡ Why the “Ready” Bits Matter

Oscillators are analog circuits.

They:

- Take time to stabilize
    
- Must reach steady oscillation
    
- Cannot be used immediately
    

The RDY bit tells you:

> “The frequency is stable. Safe to use.”

---

##  What Happens If You Skip Waiting?

If you switch clocks before RDY:

- System may freeze
    
- MCU may crash
    
- Timing becomes unpredictable
    

This is why clock configuration requires ordered steps.

---

##  Practical Clock Setup Flow

To run at 84 MHz (example):

1. Enable HSE (CR)
    
2. Wait for HSERDY
    
3. Configure PLL (PLLCFGR)
    
4. Enable PLL (CR)
    
5. Wait for PLLRDY
    
6. Switch SYSCLK to PLL (CFGR)
    

CR handles steps 1, 2, 4, 5.

---

##  Mental Model

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
