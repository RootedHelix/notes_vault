Related to [[Embedded]], [[System Clock Tree]], [[AHB Advanced High Performance Bus]]
#embedded #apb 

## APB in Embedded Systems

**APB** stands for **Advanced Peripheral Bus**.

It is a **low-power, lower-speed internal bus** used to connect the CPU to peripheral devices inside a microcontroller.

> If AHB is the highway, APB is the side street where peripherals live.

APB is part of ARM’s AMBA architecture and is used in Cortex-M MCUs like the STM32F446RE.

---

##  Where APB Fits

Inside STM32 devices, the clock flow typically looks like this:

```
PLL → SYSCLK
        ↓
      AHB (HCLK)
        ↓
   APB1      APB2
```

- **AHB** feeds the core and high-speed logic
    
- **APB1 / APB2** feed peripheral blocks
    

---

##  Internal Bus Overview


![[APB_example.jpg]]

![Image](https://d262ilb51hltx0.cloudfront.net/max/800/0%2AU-61cb7vuTjzUts5.)

![Image](https://i.sstatic.net/7OJlZ.png)

![Image](https://community.st.com/t5/image/serverpage/image-id/125204i6DBB5D7B4FD4D4C5/image-size/medium/is-moderation-mode/true?px=400&v=v2)

---

##  Why Two APBs?

In STM32F4 (like yours), there are usually:

###  APB1 (Lower Speed)

Used for:

- USART2
    
- I2C
    
- CAN
    
- Some timers
    

Max frequency is typically lower.

---

###  APB2 (Higher Speed)

Used for:

- SPI1
    
- USART1
    
- ADC
    
- Advanced timers
    

Runs faster than APB1.

---

## ⚙️ Example Clock Setup

Let’s say:

- SYSCLK = 84 MHz
    
- AHB prescaler = 1 → HCLK = 84 MHz
    
- APB1 prescaler = 2 → APB1 = 42 MHz
    
- APB2 prescaler = 1 → APB2 = 84 MHz
    

Now:

- UART on APB1 runs from 42 MHz
    
- SPI1 on APB2 runs from 84 MHz
    

---

## ⚠️ Important STM32 Detail (Very Important)

If:

```
APB prescaler > 1
```

Then:

```
Timer clock = APB clock × 2
```

This surprises many beginners.

Example:

APB1 = 42 MHz  
Timer on APB1 actually runs at **84 MHz**

This is why timers sometimes run "twice as fast" as expected.

---

## 🔌 Why APB Exists

APB is designed to:

- Be simpler than AHB
    
- Use less power
    
- Reduce hardware complexity
    
- Serve slower peripherals efficiently
    

Peripherals don’t need full CPU-speed bandwidth.

---

## 🧠 Big Picture Mental Model

Think of the MCU like this:

|Component|Role|
|---|---|
|PLL|Creates high-speed clock|
|AHB|Feeds CPU + memory|
|APB|Feeds peripherals|
|Prescalers|Control speeds|

So APB is:

> The controlled-speed distribution bus for communication peripherals.

---

## 🎯 Why This Matters for You

Since you’re working toward deeper embedded understanding:

APB affects:

- UART baud rate accuracy
    
- Timer calculations
    
- ADC timing
    
- SPI speed
    
- Interrupt frequency
    

If APB clocks are wrong → peripherals misbehave.

---

If you'd like, next we can:

- 🔹 Walk through a real timer frequency calculation
    
- 🔹 Configure clocks manually without CubeMX
    
- 🔹 Or break down the full STM32 clock tree step-by-step like an embedded engineer would
    

You’re asking the right foundational questions.