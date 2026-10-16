
Related to [[Embedded]] [[System Clock Tree]] [[HAL Library]]
#embedded #hsi 

HSI stands for High Speed Internal Oscillator which is an internal clock source built into an MCU.


> An oscillator circuit built inside the chip.

It does **not** use a crystal.

It uses an internal RC (resistor-capacitor) network that naturally charges and discharges repeatedly.

That charge/discharge cycle creates oscillation.

Think of it like:

Voltage rises → threshold reached → circuit flips → voltage falls → repeat.

It’s convenient because:

- No external components required
    
- Starts quickly
    
- Always available after reset
    

But:

- Less precise
    
- More temperature drift
    

On many STM32 parts, HSI is 16 MHz.

HSI is an **on-chip RC oscillator** that generates a clock signal without needing any external crystal.

Instead of this:

![[HSI_circuit.jpg]]

![[HSI_physical.jpg]]


you can use the **internal oscillator** that’s already inside the MCU silicon.

---

## ⚙️ How It Works

- Implemented as an **RC oscillator**
    
- Built directly into the chip
    
- Starts up immediately on power-up
    
- No external components required
    

On many STM32 MCUs (like your **STM32F446RE**):

- HSI = **16 MHz**
    
- Used as default system clock after reset
    

---

##  HSI vs HSE

|Feature|HSI|HSE|
|---|---|---|
|Meaning|High-Speed Internal|High-Speed External|
|Location|Inside MCU|External crystal|
|Accuracy|Moderate (±1% to ±5%)|Very high (±20–50 ppm typical)|
|Cost|Free|Requires crystal|
|Startup Time|Very fast|Slightly slower|
|Best For|Basic apps, dev|USB, precise timing, comms|

---

## When Do You Use HSI?

HSI is good for:

- Early bring-up
    
- LED blinking
    
- Simple GPIO work
    
- Low-cost designs
    
- When timing precision is not critical
    

You typically **do NOT** use HSI for:

- USB (needs very precise 48 MHz clock)
    
- Accurate UART at high baud
    
- RF communication
    
- Real-time clocks
    
- Audio
    

---

##  Why It Matters (Bare Metal Thinking)

Since you’re learning MCU fundamentals, this is important:

The clock source determines:

- CPU speed
    
- Timer speed
    
- UART baud rate accuracy
    
- Power consumption
    
- Peripheral timing
    

If your clock is off → everything derived from it is off.

---

## 🏗 Conceptual Clock Tree (STM32 Example)

![[clock_tree_example.jpg]]

HSI feeds into:

- PLL (if used)
    
- System clock (SYSCLK)
    
- AHB/APB buses
    
- Timers
    
- Peripherals
    

Everything starts with a clock source.

---

## Practical Example

On reset:

`SystemInit();  // Uses HSI by default`

Your MCU boots at 16 MHz from HSI unless you switch it to HSE + PLL.

---

## Simple Mental Model

Think of HSI as:

> The MCU’s built-in heartbeat.

No external parts needed — but slightly imperfect rhythm.

# Further Explanation : 

Good — this is the right question.

You understand **what** HSI is.  
Now let’s talk about **what it actually does electrically and architecturally.**

---

## What the HSI _Actually Does_

At the most fundamental level:

> **HSI generates a periodic electrical signal (a square wave) that drives every synchronous digital circuit inside the MCU.**

That’s it.

But that simple statement is extremely powerful.

---

## ⚡ Step 1 — What Is a Clock Signal?

A clock is just a voltage toggling:

```
0V → 3.3V → 0V → 3.3V → ...
```

At a fixed frequency.

If HSI = 16 MHz:

That means:

```
16,000,000 cycles per second
```

Each cycle is:

```
1 / 16,000,000 = 62.5 ns
```

Every 62.5 nanoseconds, the MCU gets a “tick.”

---

##  Step 2 — Why the CPU Needs It

The CPU is built from:

- Flip-flops
    
- Latches
    
- Registers
    
- State machines
    

These are **synchronous digital circuits**.

They change state only on a clock edge:

```
↑ Rising edge
```

Without a clock:

- The CPU cannot advance instructions
    
- Registers won’t update
    
- Timers won’t count
    
- UART won’t shift bits
    
- Nothing moves
    

The oscillator is what makes computation possible.

---

##  Inside the CPU (Conceptual)

Every instruction execution looks like:

```
Tick 1 → Fetch instruction
Tick 2 → Decode
Tick 3 → Execute
Tick 4 → Write back
```

No clock = no progression.

The clock is what advances the pipeline.

---

## ⏱ Step 3 — Timers

If you configure:

```c
TIM2->PSC = 16000 - 1;
TIM2->ARR = 1000 - 1;
```

You are dividing the clock.

If HSI = 16 MHz:

```
16,000,000 / 16,000 = 1000 Hz
```

Timer ticks at 1 kHz.

Then counts to 1000:

```
1000 Hz / 1000 = 1 Hz
```

So your LED toggles every 1 second.

Without HSI, the timer would never increment.

---

##  Step 4 — UART Example

UART baud rate depends on clock.

If you want 115200 baud:

The MCU calculates:

```
Clock / baud divisor
```

If your clock is slightly inaccurate (HSI can drift ±1% or more):

Your UART timing drifts.

At high speeds → communication errors.

This is why precision clocks matter.

---

##  Step 5 — Power Consumption

Higher clock frequency:

- More switching
    
- More current
    
- More heat
    

Lower clock:

- Less switching
    
- Lower power
    

That’s why you can downclock the MCU.

HSI directly determines:

> How fast the entire chip switches internally.

---

##  What the HSI Physically Is

It’s an **RC oscillator circuit** built in silicon.

Simplified concept:

```
Resistor + Capacitor → charging/discharging cycle → oscillation
```

![Image](https://cmds-test.vedantu.com/prod/question-sets/70702e7c-6492-45d6-9dca-61e790e4dac88707625875449304569.png)

![[HSI_oscillator.jpg]]

![Image](https://i.sstatic.net/S5fGI.jpg)

![Image](https://www.allaboutcircuits.com/uploads/articles/square_wave_generator_figure_2.png)

This oscillation gets squared up into a digital clock.

---

## So What Does HSI _Do_?

It:

1. Creates the base timing reference
    
2. Drives CPU instruction timing
    
3. Feeds timers
    
4. Sets communication timing
    
5. Determines peripheral speed
    
6. Determines power usage
    

It is literally the tempo of the entire system.

---

## Deep Mental Model

Think of the MCU as:

- Millions of tiny switches
    
- That only change position on clock edges
    

HSI is the metronome.

No metronome → frozen orchestra.

---

##  Why This Matters For You (STM32 Learning)

When you switch:

HSI → HSE → PLL

You are changing:

- CPU speed
    
- Peripheral timing
    
- Power profile
    
- Stability
    
- Communication reliability
    

Clock configuration is not just “frequency.”

It is system architecture.

---
