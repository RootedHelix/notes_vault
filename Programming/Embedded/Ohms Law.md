Related to [[Embedded]], [[Circuit]], [[Resistor]], [[System Clock Tree]]
#embedded #ohms_law #current 

## ⚡ Ohm’s Law

**Ohm’s Law** describes the relationship between:

- **Voltage (V)** – electrical pressure
    
- **Current (I)** – flow of electric charge
    
- **Resistance (R)** – opposition to current flow
    

The formula is:


`V = I x R`


That’s it. That’s the foundation of almost all low-level embedded electrical reasoning.

---

## 🧠 What Each Term Means (Embedded Context)

### Voltage (V)

Think of voltage as **electrical pressure**.

In your STM32F446RE system, common voltages are:

- 3.3V (MCU logic level)
    
- 5V (USB or some peripherals)
    

---

### Current (I)

Current is how much charge flows per second.

Measured in:

- Amps (A)
    
- Milliamps (mA) → 0.001 A
    

MCU GPIO pins usually handle:

- ~10–20 mA safely (check datasheet)
    

---

### Resistance (R)

Measured in **Ohms (Ω)**.

A resistor controls how much current flows for a given voltage.

---

##  The Three Forms of Ohm’s Law

You rearrange depending on what you need:

`V = I x R`

 
`I = V / R`  

  
`R = V / I`


---

##  Practical Embedded Example (LED on GPIO)

Let’s say:

- GPIO output = 3.3V
    
- LED drop = 2.0V
    
- Desired current = 10 mA (0.01 A)
    

Voltage across resistor:

  
`3.3 - 2.0 = 1.3V`


Now solve for R:

  
`R = (1.3 / 0.01) = 130Ω`


So you’d use ~150Ω.

That calculation is **pure Ohm’s Law**.

---

# 🧲 Why This Matters in Embedded Systems

Ohm’s Law lets you:

- Prevent frying MCU pins
    
- Size resistors for LEDs
    
- Calculate pull-up resistor current
    
- Design voltage dividers
    
- Estimate power usage
    

If you understand Ohm’s Law deeply, hardware stops feeling mysterious.

---

# ⚙️ Intuitive Way to Think About It

Water analogy:

- Voltage = water pressure
    
- Current = water flow rate
    
- Resistance = pipe restriction
    

More resistance → less flow  
More voltage → more flow  
More resistance → requires more voltage to maintain same current

---

# 🔥 One More Important Formula (Power)

Often paired with Ohm’s Law:


`P = V \times I`


Using substitution:


`P = I^2 R`  


 
`P = V^2 / R`


This tells you:

- How much heat a resistor dissipates
    
- Why resistors have power ratings (¼W, ½W, etc.)
    

---

If you'd like, I can now explain:

- Why current increases when resistance decreases (physically)
    
- How this applies to STM32 GPIO damage
    
- How to calculate pull-up resistor values properly
    
- Or how Ohm’s Law ties into clock circuits and oscillators
    

What direction do you want to go deeper in?

