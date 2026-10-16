Related to [[Emvedded]], [[Circuit]], [[Inductor]], [[GPIO Registers]]
#embedded #resistor #circuit 

A **resistor** in embedded systems is one of the most fundamental electronic components. It is a passive component that **limits or controls the flow of electrical current** in a circuit.

At the physics level:

> A resistor converts electrical energy into heat and creates a predictable voltage drop.

---

##  What It Actually Does

A resistor follows **Ohm’s Law**:

  
 `V = I × R` 


Where:

- **V** = voltage (volts)
    
- **I** = current (amps)
    
- **R** = resistance (ohms Ω)
    

So if you know two, you can calculate the third.

---

# 🧠 Why Resistors Matter in Embedded Systems

In microcontroller systems (like your STM32F446RE), resistors are used constantly for:

---

## 1️⃣ Limiting Current (Protecting Pins & LEDs)

![Image](https://eepower.com/uploads/articles/resistors-in-LED-circuits.png)

![Image](https://images.openai.com/static-rsc-3/q39VajiYlAH2pu5LoJkIHsS1F8ZDk7BKX23ONNZxXEkXIuK14DYjo5cBMZq7gp9PSXIhBBHkRJngRReN4kpmgQYPDVHx5bP1mGGgQ5IpqzA?purpose=fullsize&v=1)

![Image](https://i.sstatic.net/vfoo1.png)

![Image](https://i.sstatic.net/aX6S9.png)

MCU GPIO pins cannot source unlimited current.

If you connect an LED directly to a pin:

- Too much current flows
    
- The LED or MCU pin can burn out
    

So you add a resistor in series to **limit current**.

Example:

- MCU output = 3.3V
    
- LED drop ≈ 2.0V
    
- Desired current = 10mA
    

[  
R = \frac{3.3 - 2.0}{0.01} = 130Ω  
]

You’d typically use 150Ω.

This is probably the most common beginner use case.

---

## 2️⃣ Pull-Up and Pull-Down Resistors (Stabilizing Inputs)

![Image](https://cdn.sparkfun.com/assets/6/f/b/c/7/511568b6ce395f1b40000000.jpg)

![Image](https://europe1.discourse-cdn.com/arduino/original/4X/f/3/7/f373a4b793d353303484c14b2b9c663dfef2f16c.jpeg)

![Image](https://www.mouser.com/blog/Portals/11/MikeParksPhotos/Pins1.png)

![Image](https://www.labcenter.com/blog/sim-inputs-outputs/images/xpullUpResistor.png.pagespeed.ic.dp1VVmB84R.png)

When you configure a GPIO pin as input, it can “float.”

Floating means:

- It randomly reads HIGH or LOW due to noise.
    
- It is electrically unstable.
    

So we add:

- **Pull-up resistor** → connects input to Vcc
    
- **Pull-down resistor** → connects input to GND
    

This ensures a **default known state**.

Good news:  
Your STM32 has **internal pull-ups and pull-downs** you can enable in software.

---

## 3️⃣ Voltage Dividers

![Image](https://www.allaboutcircuits.com/uploads/thumbnails/voltage-divider-main-circuit.png)

![Image](https://www.researchgate.net/publication/336424692/figure/fig1/AS%3A812806827937793%401570799657755/Scheme-of-a-two-resistor-voltage-divider.ppm)

![Image](https://i.sstatic.net/pucoT.png)

![Image](https://i.sstatic.net/Z96uK.png)

Two resistors can scale voltage down.

Example:

You want to measure 5V with a 3.3V ADC pin.

Use a divider:

[  
V_{out} = V_{in} \times \frac{R2}{R1 + R2}  
]

This is common for:

- Battery measurement
    
- Sensor scaling
    
- Level shifting (carefully)
    

---

# 🔍 What a Resistor Physically Is

![Image](https://th.element14.com/productimages/large/en_GB/9338187-40.jpg)

![Image](https://thesaberarmory.com/cdn/shop/products/0603_887b298e-26c7-49b6-bfb9-b35841e6cae8_360x.jpg?v=1589735825)

![Image](https://images.openai.com/static-rsc-3/d-iv3lr83CDQWatiRG0al_xi2YekQZqwiMpAdcOD07cl1aKQPsEm7ERMW6Sgf9JQ2kzXxQBlqSNoykXfEQmx7c_xj-58p5YEt2Jb-Oc6iSk?purpose=fullsize&v=1)

![Image](https://images.openai.com/static-rsc-3/c5H2XWzi3qqHatHOsY9KHNBLz3v1zLQ04MvFhDK4YZ1eniUnpl9fFdoL0tv9tj3ztoKQ6ewUrW8nb7GOuy8FNalg3qLWZq5EHCesWXqi0PQ?purpose=fullsize&v=1)

They come in:

- Through-hole (color bands)
    
- Surface-mount (tiny rectangles)
    
- Different power ratings (¼W, ½W, etc.)
    
- Different tolerance (1%, 5%)
    

---

# ⚙️ In Embedded Thinking Terms

Resistors are not “logic.”

They are **analog control tools**.

They:

- Protect hardware
    
- Shape voltage levels
    
- Set bias conditions
    
- Define timing (with capacitors)
    
- Set current levels
    

They are part of the **hardware side of firmware engineering**.

---

# 🚨 Important Embedded Concept

Microcontroller pins are:

- Voltage-controlled inputs
    
- Current-limited outputs
    

Resistors allow you to:

- Match electrical realities to digital expectations
    

Without them, embedded systems would be unstable or destroyed.

---

If you'd like, I can next explain:

- Why floating inputs physically oscillate
    
- How internal pull-ups are implemented inside the MCU
    
- How resistors interact with capacitors (RC timing)
    
- How resistor values affect power consumption
    
- How to choose resistor values professionally
    

Where do you want to go deeper?