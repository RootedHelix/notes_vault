Related to [[Embedded]], [[Resistor]], [[Ohms Law]], [[Inductor]]
#embedded #resistor #inductor

## ⚡ What Is a Capacitor? (Embedded Systems Context)

A **capacitor** is a passive electronic component that **stores electrical energy in an electric field**.

If a resistor controls current, a capacitor controls **how voltage changes over time**.

---

## 🧠 The Core Idea

A capacitor resists _changes in voltage_.

- Resistor → resists current
    
- Capacitor → resists voltage change
    

That difference is huge in embedded systems.

---

## 🔋 What It Physically Is

![Image](https://www.sparkfun.com/media/catalog/product/cache/a793f13fd3d678cea13d28206895ba0c/0/8/08375-03-L.jpg)

![[capacitor_physically.jpg]]

![[capacitor_anatomy.jpg]]

![Image](https://images.openai.com/static-rsc-3/aNF-I5b1k2GTWRBFYltmepePlDTIX3IJUbbjw35OW3_oQ5M05eQQILxtDLL1upsR1-lJmpq9Dl__AiPN7hSBTKD4fCFfcQonQlbX2B1qFNA?purpose=fullsize&v=1)

Inside a capacitor:

- Two conductive plates
    
- Separated by an insulator (dielectric)
    

When voltage is applied:

- Electrons accumulate on one plate
    
- Opposite charge forms on the other
    
- Energy is stored in the electric field between them
    

---

## 📐 The Fundamental Equation

  
`Q = C x V`


Where:

- **Q** = stored charge (Coulombs)
    
- **C** = capacitance (Farads)
    
- **V** = voltage
    

Capacitance is measured in:

- Farads (F)
    
- Microfarads (µF)
    
- Nanofarads (nF)
    
- Picofarads (pF)
    

In embedded systems, you’ll mostly see:

- 0.1µF
    
- 1µF
    
- 10µF
    
- 100µF
    

---

# 🔌 Why Capacitors Matter in Embedded Systems

Using your STM32F446RE as an example:

---

## 1️⃣ Decoupling / Bypass Capacitors (Most Important Use)



![[decouple_capacitor_1.jpg]]
![[decouple_capacitor_2.jpg]]

![Image](https://i.sstatic.net/SuiNB.png)

MCUs draw tiny bursts of current when switching internally.

That causes voltage dips.

A small capacitor near the VCC pin:

- Supplies instant current
    
- Smooths voltage spikes
    
- Reduces noise
    

Without these → MCU instability.

That little 0.1µF capacitor next to the chip?  
Extremely important.

---

## 2️⃣ Power Supply Filtering


![[power_ripple.jpg]]
![[power_ripple_2.jpg]]
![Image](https://i.sstatic.net/pKqZu.png)


Capacitors smooth ripple in power supplies.

They:

- Charge when voltage rises
    
- Discharge when voltage falls
    
- Reduce noise
    

---

## 3️⃣ Timing (RC Circuits)

![Image](https://www.allaboutcircuits.com/uploads/articles/RC-circuit-discharge-curve.jpg)

![[capacitor_image.jpg]]


When paired with a resistor:

You get a time delay.

The time constant:


`tau  (T) = R  x C`


This determines how fast voltage rises or falls.

Used for:

- Reset circuits
    
- Debouncing buttons
    
- Simple oscillators
    
- Analog filtering
    

---

## 4️⃣ Signal Filtering

Capacitors help:

- Remove high-frequency noise
    
- Smooth ADC readings
    
- Shape signals
    

They are essential in:

- Analog circuits
    
- Clock circuits
    
- Communication lines
    

---

# ⚙️ Intuitive Mental Model

Water analogy again:

- Resistor = narrow pipe
    
- Capacitor = flexible balloon attached to pipe
    

When pressure spikes:

- Balloon absorbs the surge
    
- Releases it when pressure drops
    

That’s voltage smoothing.

---

# 🔥 Important Embedded Insight

Capacitors are about **time behavior**.

Resistors define:

- How much current
    

Capacitors define:

- How fast voltage changes
    

Together they create dynamic behavior.

---

# 🚨 Practical Embedded Reality

If you build a board and forget decoupling capacitors:

- Random resets
    
- ADC noise
    
- Communication glitches
    
- Unstable clocks
    

Capacitors are not optional decoration.  
They are stability components.

---

If you'd like, next we can go deeper into:

- Why capacitors block DC but pass AC
    
- How they interact with PLL and oscillators
    
- Why decoupling caps must be physically close to MCU pins
    
- How RC timing creates exponential curves
    
- How button debouncing works with an RC filter
    

Where do you want to dig in next?