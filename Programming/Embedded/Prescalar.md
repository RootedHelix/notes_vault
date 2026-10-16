Related to [[Embedded]], [[System Clock Tree]], [[PLL Phase Locked Loops]] 
#embedded #prescalar_embedded

## What Does a Prescaler Do?

A **prescaler** is a hardware divider that **reduces a clock frequency** before passing it to another part of the system.

In simple terms:

> 🔢 It divides the clock by a fixed number.

---

## ⚙️ Basic Formula

If:

```
Input Clock = 84 MHz
Prescaler = 4
```

Then:

```
Output Clock = 84 MHz / 4 = 21 MHz
```

That’s it. It slows the clock down.

---

## 🏗 Where Prescalers Exist (STM32 Example)

In microcontrollers like the STM32F446RE, prescalers are everywhere:

- AHB prescaler
    
- APB1 prescaler
    
- APB2 prescaler
    
- Timer prescaler
    
- ADC prescaler
    

They control how fast different parts of the MCU run.

---

## 🖼 Clock Tree Context

![Image](https://community.st.com/t5/image/serverpage/image-id/122181i39714574C135C159?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/104755iF2DCC38F8F0D2859/image-size/large/is-moderation-mode/true?px=999&v=v2)

![Image](https://www.digikey.com/maker-media/cc92068e-069c-4bdf-8b5e-157414a854f2)

![Image](https://www.digikey.com/maker-media/56a115f4-087b-4bfd-b897-feaae9756db9)

---

## 🧩 Why Prescalers Are Needed

Not every component should run at full CPU speed.

Reasons:

### 🔹 1. Power Control

Slower clock = less power consumption.

### 🔹 2. Peripheral Limits

Some peripherals have maximum clock limits.

Example:

- APB1 might max at 45 MHz
    
- CPU might run at 180 MHz
    

So we divide it.

### 🔹 3. Timing Precision

Timers need slower base clocks to generate usable time intervals.

---

## 🧠 Most Important Example (Timers)

Let’s say:

- Timer clock = 84 MHz
    
- Timer prescaler = 8399
    

Formula:

```
Timer Frequency = 84 MHz / (Prescaler + 1)
```

So:

```
84,000,000 / 8400 = 10,000 Hz
```

Now your timer ticks every:

```
1 / 10,000 = 0.0001 seconds = 100 µs
```

Now you can build precise millisecond delays.

---

## 🔬 What a Prescaler Physically Is

In hardware, it’s usually:

- A binary counter
    
- That only passes a pulse every N cycles
    

Example:

Divide-by-4 prescaler:

```
Clock in:  |‾|_|‾|_|‾|_|‾|_|‾|_
Output:    |‾‾‾‾|____|‾‾‾‾|____
```

It outputs one pulse for every 4 input pulses.

---

## 🧠 Mental Model

Think of it like gears:

Big fast gear → small slower gear

Or like this:

> PLL makes clocks fast  
> Prescalers tame them down

---

## ⚠️ Subtle STM32 Detail

For STM32 timers:

```
Actual timer frequency = 
    TimerClock / (PSC + 1)
```

That `+1` matters.

Many beginners forget it and their timing is slightly wrong.

---

## 🎯 Why This Matters For You

As you move into:

- Bare-metal timers
    
- Interrupt frequency setup
    
- PWM generation
    
- UART baud calculation
    

Prescalers are critical.

They are how you turn:

> 84 MHz into 1 second.

---

## 🧠 Big Insight

PLL lets you generate a **high master frequency**.

Prescalers let you distribute **controlled speeds** across subsystems.

Without PLL → system too slow  
Without prescalers → peripherals overclock or waste power

---

##  The Clean Mental Model

Think of it like this:

- PLL = Turbocharger
    
- Prescaler = Gearbox
    

Turbo makes power.  
Gearbox controls how that power is delivered.

---