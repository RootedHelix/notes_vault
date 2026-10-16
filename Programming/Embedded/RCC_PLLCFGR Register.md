Related to  [[Embedded]]

Stems from [[RCC Reset Clock Control]]

#rcc_pllcfgr_register

## What is the Purpose of this Register?

On STM32 (like your F446RE), the  
**`RCC_PLLCFGR` register** configures the **PLL (Phase Locked Loop)**.

The PLL is what allows you to multiply a clock source (HSI or HSE) up to a higher system frequency like 84 MHz, 100 MHz, 168 MHz, etc.

It configures:

1. Which clock feeds the PLL
    
2. How much to multiply it
    
3. How much to divide it
    
4.  What the PLL outputs are
    

In short:

> It defines how you turn 8 MHz or 16 MHz into your final SYSCLK.

# 📊 Conceptual Flow

Example:

HSE = 8 MHz  
PLL multiplies → 168 MHz  
SYSCLK = 168 MHz

All of that math is defined inside `PLLCFGR`.

---

# 🔎 The Fields Inside PLLCFGR (STM32F4)

Here are the important ones:

|Field|Purpose|
|---|---|
|**PLLSRC**|Selects HSI or HSE|
|**PLLM**|Input divider|
|**PLLN**|Main multiplier|
|**PLLP**|System clock divider|
|**PLLQ**|USB / peripheral divider|

---

# 🧮 How The Math Works

The formula:

$$
\begin{aligned}
PLL_{VCO} &= \frac{Input}{PLLM} \times PLLN \\\\
SYSCLK &= \frac{PLL_{VCO}}{PLLP} \\\\
USB &= \frac{PLL_{VCO}}{PLLQ}
\end{aligned}
$$
---

# 🧪 Real Example (STM32F446)

Suppose:

- HSE = 8 MHz
    
- PLLM = 8
    
- PLLN = 336
    
- PLLP = 2
    

Step 1:

8MHz/8=1MHz8MHz / 8 = 1MHz8MHz/8=1MHz

Step 2:

1MHz×336=336MHz(VCO)1MHz \times 336 = 336MHz (VCO)1MHz×336=336MHz(VCO)

Step 3:

336MHz/2=168MHzSYSCLK336MHz / 2 = 168MHz SYSCLK336MHz/2=168MHzSYSCLK

That entire configuration is written into `RCC->PLLCFGR`.

---

# 🛠 Where You Use It

When you manually configure clocks (not CubeMX), you do:

`RCC->PLLCFGR = ...`

But important:

1. Disable PLL
    
2. Configure PLLCFGR
    
3. Enable PLL
    
4. Wait for PLL ready
    
5. Switch SYSCLK to PLL
    

---

# 🚨 Why This Register Is Critical

If you misconfigure it:

- MCU may not boot
    
- USB won’t work
    
- Flash wait states will be wrong
    
- System becomes unstable
    

This is why embedded engineers must truly understand this register.

---

# 🧩 How This Connects to Your Learning

Since you're manually working through:

- RCC->CR
    
- RCC->CFGR
    
- RCC->PLLCFGR
    

This is exactly the right step.

You’re no longer just “using HAL” — you’re controlling the silicon.