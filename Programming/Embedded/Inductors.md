Related to [[Embedded]], [[Current]], [[Resistor]], [[Circuit]]
#embedded #inductor #current 

## Inductor Definition
An **inductor** in embedded systems is a **passive electronic component that stores energy in a magnetic field when current flows through it**.

In simple terms:

>  A capacitor resists changes in **voltage**  
>  **An inductor resists changes in current**

---

## What It Looks Like

![[Inductor_image_1.jpg]]

![[Inductor_image_2.jpg]]

![[Inductor_image_3.jpg]]
![[Inductor_image_4.jpg]]
Physically, an inductor is just:

- A coil of wire
    
- Sometimes wrapped around a magnetic core (ferrite or iron)
    
- Can be through-hole or tiny SMD on a PCB
    

---

## The Core Idea

When current tries to change quickly, the inductor pushes back.

Mathematically:

  
`V = L (dI / dt)`


Where:

- **L** = inductance (Henries)
    
- **dI/dt** = rate of change of current
    

So:

- If current changes fast → voltage spike
    
- If current is steady → behaves like a wire
    

---

# Why Inductors Matter in Embedded Systems

You don’t usually use inductors directly in firmware…

You encounter them in **power electronics and signal conditioning**.

---

## 1️⃣ Power Regulation (Very Important)

Inside:

- Switching regulators
    
- Buck converters
    
- Boost converters
    

Every STM32 board (like your F446RE) has inductors in its power section.

Example:

- USB gives 5V
    
- MCU needs 3.3V
    
- A switching regulator uses an **inductor** to step voltage down efficiently.
    

Without inductors:

- You’d waste power as heat (linear regulators).
    

---

## 2️⃣ Filtering Noise

Inductors + capacitors form:

- LC filters
    
- EMI suppression circuits
    

They smooth noisy signals before they reach:

- ADC inputs
    
- RF modules
    
- Sensitive analog circuits
    

---

## 3️⃣ Energy Storage for Switching

In switching supplies:

- Transistor turns ON → current builds in inductor (stores energy)
    
- Transistor turns OFF → inductor releases energy
    

This is how efficient DC-DC conversion works.

---

## 4️⃣ Motors & Actuators

A motor winding **is basically an inductor**.

So when you:

- Drive a relay
    
- Control a motor
    
- Switch a solenoid
    

You are dealing with inductive loads.

That’s why we use:

- Flyback diodes
    
- Snubber circuits
    

---

# Conceptual Intuition

Think of it like:

|Component|Resists Change In|Stores Energy In|
|---|---|---|
|Resistor|Nothing (dissipates)|Heat|
|Capacitor|Voltage|Electric field|
|Inductor|Current|Magnetic field|

---

# Where This Connects to Your STM32 Learning

As you move into:

- ADC
    
- PWM motor control
    
- Switching power
    
- RF modules
    
- Embedded hardware design
    

Understanding inductors becomes critical.

Especially when you:

- See voltage spikes
    
- Blow a MOSFET
    
- Wonder why your signal is noisy
    
- Build a motion sensor with power regulation
    

---

If you'd like, I can next explain:

- 🔋 How a buck converter actually works step-by-step
    
- ⚡ Why inductors cause voltage spikes
    
- 🧲 How inductance is measured and chosen
    
- 🛠 How this applies to STM32 board design
    

Which direction do you want to go?