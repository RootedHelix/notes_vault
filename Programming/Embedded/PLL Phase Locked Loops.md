##  PLL in Embedded Systems (Phase-Locked Loop)

A **PLL (Phase-Locked Loop)** is a hardware circuit used to **multiply and stabilize clock frequencies** inside a microcontroller or processor.

In simple terms:

> It takes a lower-frequency clock (like 8 MHz) and generates a higher, stable clock (like 84 MHz or 168 MHz).

This is extremely important in embedded systems because **everything runs on the system clock** — CPU, timers, UART, SPI, ADC, etc.

---

##  Why PLL Exists

Microcontrollers usually have:

- An internal oscillator (HSI – High-Speed Internal)
    
- Or an external crystal (HSE – High-Speed External)
    

But these raw clocks are often too slow for full performance.

So we use the PLL to:

-  Increase frequency
    
-  Precisely tune frequency
    
-  Lock phase for stability
    

---

## ⚙️ Conceptual Diagram

![Image](https://www.tutorialspoint.com/linear_integrated_circuits_applications/images/block_diagram_of_pll.jpg)

![Image](https://www.analog.com/en/_/media/images/analog-dialogue/en/volume-52/number-3/articles/phase-locked-loop-pll-fundamentals/184330_fig_01.png?la=en&rev=c1086cbd04c040abb7b5caffa6cc8824&sc_lang=en)

![Image](https://community.st.com/t5/image/serverpage/image-id/57651i8E58C576320D40EA?v=v2)

![Image](https://community.st.com/t5/image/serverpage/image-id/61919iFB739101EBD93CBA?v=v2)

---

##  How a PLL Works (Simplified)

A PLL has three main parts:

### 1️ Phase Detector

Compares:

- Input clock
    
- Feedback clock (from output)
    

It measures phase difference.

---

### 2️ VCO (Voltage-Controlled Oscillator)

Adjusts its frequency based on the phase difference.

---

### 3️ Feedback Divider

Divides the output clock and feeds it back to the phase detector.

The loop keeps adjusting until:

>  Output phase = Input phase (locked)

That’s why it’s called a **Phase-Locked Loop**.

---

##  Example in STM32 (Like Your Nucleo F446RE)

In the STM32F446RE:

You might have:

- HSE = 8 MHz crystal
    
- PLL multiplies it to 84 MHz or 180 MHz
    

Example configuration conceptually:

```
8 MHz (HSE)
   ↓
PLL
   ↓
84 MHz System Clock (SYSCLK)
```

Now your CPU runs 10× faster.

---

##  Why This Matters for You

Since you're working toward bare-metal understanding:

When you configure clocks in STM32CubeMX, you're actually:

- Selecting clock source (HSI or HSE)
    
- Configuring PLL multipliers/dividers
    
- Setting SYSCLK
    
- Configuring AHB/APB bus speeds
    

Understanding PLL means you understand:

- Why timers tick at certain speeds
    
- Why UART baud rate works or fails
    
- Why USB requires exactly 48 MHz
    
- Why ADC sampling depends on clock prescalers
    

---

##  Real Example

If you set:

- HSE = 8 MHz
    
- PLL multiplier = 21
    
- PLL divider = 2
    

Then:

```
8 MHz × 21 / 2 = 84 MHz
```

That becomes your system clock.

---

## ⚠️ Important Notes

- PLL must stabilize before use
    
- Some peripherals require specific derived clocks
    
- Too high frequency = higher power consumption
    
- Clock tree configuration directly affects interrupt timing
    

---

##  Big Picture

PLL is part of the **clock tree** of a microcontroller.

Without a properly configured PLL:

- Your MCU runs slow
    
- Peripherals misbehave
    
- Timing calculations break
    

---
