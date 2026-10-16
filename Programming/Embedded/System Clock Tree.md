Related to [[Embedded]], [[HAL Library]]
#embedded #system_clock_tree

## Global CPU + Bus clocks

This is what:

`SystemClock_Config();`

does.

It configures:

- HSI vs HSE
    
- PLL multipliers/dividers
    
- SYSCLK
    
- AHB clock
    
- APB1 / APB2 clocks
    
- Flash latency
    

This determines:

- CPU frequency
    
- Timer base speeds
    
- Peripheral bus speeds
    
- Overall system timing
    

Without this, your MCU just runs at its reset default (often HSI 16 MHz on STM32F4).

**Why is the System Clock Tree Separate from the Peripheral Clocks**

Because they are fundamentally different responsibilities:

| SystemClock_Config() | HAL Peripheral Init           |
| -------------------- | ----------------------------- |
| Configures PLL       | Enables peripheral clock gate |
| Sets CPU frequency   | Configures that peripheral    |
| Sets bus dividers    | Sets GPIO/UART/TIM registers  |
| Global effect        | Local effect                  |

Perfect. Now we’re getting into **real embedded engineering territory**.

Let’s walk through the clock tree of the **STM32F446RE** step-by-step the way firmware engineers actually think about it.

---

#  Big Picture: What Is the Clock Tree?

The clock tree distributes timing signals from a **source** to:

- CPU core
    
- AHB bus
    
- APB buses
    
- Timers
    
- USB
    
- ADC
    
- etc.
    

Everything in a microcontroller runs off clocks.

No clock → no activity.

---

#  Step 1 — Clock Sources (The Roots)

![Image](https://community.st.com/t5/image/serverpage/image-id/57651i8E58C576320D40EA?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/83435i2CC7899B9A0D0747?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/57655iCEF4A5399B5A506C?v=v2)

![Image](https://waijung1.aimagin.com/default8mhz.png)

You start with raw sources:

### HSI

- Internal oscillator
    
- 16 MHz on F446
    
- Less accurate
    
- No external components required
    

### HSE

- External crystal (usually 8 MHz on Nucleo boards)
    
- More accurate
    
- Required for USB / precise timing
    

###  PLL

Phase-Locked Loop

- Multiplies frequency
    
- Converts 8 MHz → 84 MHz (for example)
    

Think of PLL as a frequency gearbox.

---

# ⚙️ Step 2 — PLL (Frequency Engine)

Example configuration:

```
HSE = 8 MHz
PLL Multiplier = x168
PLL Divider = /2
```

Result:

```
(8 MHz × 168) / 2 = 84 MHz
```

That becomes:

```
SYSCLK
```

The main system clock.

---

# 🧠 Step 3 — SYSCLK (Core Clock)

SYSCLK feeds:

- Cortex-M4 CPU
    
- AHB prescaler
    

On STM32F446:

- Max SYSCLK = 180 MHz (depending on config)
    
- Common config = 84 MHz
    

This determines:

- Instruction speed
    
- Interrupt latency
    
- Overall system performance
    

---

# 🛣️ Step 4 — AHB Bus

SYSCLK → AHB

AHB feeds:

- SRAM
    
- DMA
    
- GPIO
    
- APB buses
    

You can divide it:

```
AHB = SYSCLK / 1
```

(or /2, /4, etc.)

Most projects keep AHB = SYSCLK.

---

# 🚌 Step 5 — APB1 and APB2

From AHB, clocks split:

### 🔹 APB1 (Low-speed peripherals)

- USART2
    
- I2C
    
- TIM2–7
    
- Power controller
    

Max: 45 MHz (on many F4 configs)

### 🔹 APB2 (High-speed peripherals)

- USART1
    
- ADC
    
- SPI1
    
- TIM1, TIM8
    

Max: 90 MHz typically

---

# ⚡ The Important Timer Trick

If APB prescaler > 1:

Timers get multiplied by 2.

Example:

```
APB1 = 42 MHz
Timer clock = 84 MHz
```

This is one of the most confusing STM32 behaviors.

Many beginners calculate timer periods wrong because of this.

---

# 🎯 Example Real Configuration (Common Setup)

```
HSE = 8 MHz
PLL → 84 MHz SYSCLK
AHB = 84 MHz
APB1 = 42 MHz
APB2 = 84 MHz
```

So:

- CPU = 84 MHz
    
- TIM2 (APB1 timer) = 84 MHz
    
- USART2 (APB1 peripheral) = 42 MHz
    

Notice:  
Same bus → different timer frequency.

This matters for debounce timers, PWM, etc.

---

# 🧠 Why This Matters For You

Since you’re working on:

- Non-blocking timers
    
- Interrupt-driven firmware
    
- Real-time thinking
    

You must understand:

```
Timer frequency = APB clock × multiplier
```

or your math will silently break.

---

# 🔥 What `SystemClock_Config()` Actually Does

Inside it, CubeMX calls:

```c
HAL_RCC_OscConfig();
HAL_RCC_ClockConfig();
```

Which:

1. Turns on HSE
    
2. Configures PLL
    
3. Sets flash latency
    
4. Sets prescalers
    
5. Switches SYSCLK source
    

It is literally constructing the tree.

---

# 🧠 Mental Model

Think of it like plumbing:

```
Water source (HSI/HSE)
        ↓
Pressure pump (PLL)
        ↓
Main pipe (SYSCLK)
        ↓
Branch pipes (AHB)
        ↓
Smaller branches (APB1 / APB2)
        ↓
Fixtures (Timers, UART, ADC)
```

---

#  Embedded Engineer Test

To truly understand this, try:

1. Set SYSCLK to 16 MHz (HSI only)
    
2. Measure LED toggle period with timer
    
3. Switch to 84 MHz PLL
    
4. Measure again
    

When you see timing change, it clicks permanently.

---


# Further Explanation 

Perfect. Now we’re getting into **real embedded engineering territory**.

Let’s walk through the clock tree of the **STM32F446RE** step-by-step the way firmware engineers actually think about it.

---

## Big Picture: What Is the Clock Tree?

The clock tree distributes timing signals from a **source** to:

- CPU core
    
- AHB bus
    
- APB buses
    
- Timers
    
- USB
    
- ADC
    
- etc.
    

Everything in a microcontroller runs off clocks.

No clock → no activity.

---

## Step 1 — Clock Sources (The Roots)

![Image](https://community.st.com/t5/image/serverpage/image-id/57651i8E58C576320D40EA?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/83435i2CC7899B9A0D0747?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/57655iCEF4A5399B5A506C?v=v2)

![Image](https://waijung1.aimagin.com/default8mhz.png)

You start with raw sources:

####  HSI

- Internal oscillator
    
- 16 MHz on F446
    
- Less accurate
    
- No external components required
    

####  HSE

- External crystal (usually 8 MHz on Nucleo boards)
    
- More accurate
    
- Required for USB / precise timing
    

####  PLL

Phase-Locked Loop

- Multiplies frequency
    
- Converts 8 MHz → 84 MHz (for example)
    

Think of PLL as a frequency gearbox.

---

## ⚙️ Step 2 — PLL (Frequency Engine)

Example configuration:

```
HSE = 8 MHz
PLL Multiplier = x168
PLL Divider = /2
```

Result:

```
(8 MHz × 168) / 2 = 84 MHz
```

That becomes:

```
SYSCLK
```

The main system clock.

---

## Step 3 — SYSCLK (Core Clock)

SYSCLK feeds:

- Cortex-M4 CPU
    
- AHB prescaler
    

On STM32F446:

- Max SYSCLK = 180 MHz (depending on config)
    
- Common config = 84 MHz
    

This determines:

- Instruction speed
    
- Interrupt latency
    
- Overall system performance
    

---

## Step 4 — AHB Bus

SYSCLK → AHB

AHB feeds:

- SRAM
    
- DMA
    
- GPIO
    
- APB buses
    

You can divide it:

```
AHB = SYSCLK / 1
```

(or /2, /4, etc.)

Most projects keep AHB = SYSCLK.

---

## Step 5 — APB1 and APB2

From AHB, clocks split:

#### APB1 (Low-speed peripherals)

- USART2
    
- I2C
    
- TIM2–7
    
- Power controller
    

Max: 45 MHz (on many F4 configs)

####  APB2 (High-speed peripherals)

- USART1
    
- ADC
    
- SPI1
    
- TIM1, TIM8
    

Max: 90 MHz typically

---

## ⚡ The Important Timer Trick

If APB prescaler > 1:

Timers get multiplied by 2.

Example:

```
APB1 = 42 MHz
Timer clock = 84 MHz
```

This is one of the most confusing STM32 behaviors.

Many beginners calculate timer periods wrong because of this.

---

##  Example Real Configuration (Common Setup)

```
HSE = 8 MHz
PLL → 84 MHz SYSCLK
AHB = 84 MHz
APB1 = 42 MHz
APB2 = 84 MHz
```

So:

- CPU = 84 MHz
    
- TIM2 (APB1 timer) = 84 MHz
    
- USART2 (APB1 peripheral) = 42 MHz
    

Notice:  
Same bus → different timer frequency.

This matters for debounce timers, PWM, etc.

---

##  Why This Matters For You

Since you’re working on:

- Non-blocking timers
    
- Interrupt-driven firmware
    
- Real-time thinking
    

You must understand:

```
Timer frequency = APB clock × multiplier
```

or your math will silently break.

---

##  What `SystemClock_Config()` Actually Does

Inside it, CubeMX calls:

```c
HAL_RCC_OscConfig();
HAL_RCC_ClockConfig();
```

Which:

1. Turns on HSE
    
2. Configures PLL
    
3. Sets flash latency
    
4. Sets prescalers
    
5. Switches SYSCLK source
    

It is literally constructing the tree.

---

## Mental Model

Think of it like plumbing:

```
Water source (HSI/HSE)
        ↓
Pressure pump (PLL)
        ↓
Main pipe (SYSCLK)
        ↓
Branch pipes (AHB)
        ↓
Smaller branches (APB1 / APB2)
        ↓
Fixtures (Timers, UART, ADC)
```

---

##  Embedded Engineer Test

To truly understand this, try:

1. Set SYSCLK to 16 MHz (HSI only)
    
2. Measure LED toggle period with timer
    
3. Switch to 84 MHz PLL
    
4. Measure again
    

When you see timing change, it clicks permanently.

---

If you want, next I can:

- Walk through a real timer frequency calculation step-by-step
    
- Explain flash latency and why it changes with SYSCLK
    
- ⚡ Or explain how USB requires a precise 48 MHz clock
    

Which one?