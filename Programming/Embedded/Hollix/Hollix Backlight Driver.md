Related to [[Hollix]], [[Embedded]]
#hollix_backlightdriver #hollix

# 🧠 3️⃣ What Type of Driver You Need

You need:

### ✅ Constant-current boost LED driver

### ✅ Input 2.7–5 V

### ✅ Output up to 20–24 V

### ✅ PWM dimming input

### ✅ ~40 mA output capability

Not a generic boost converter.

Not a resistor.

Not a simple step-up module.

It must regulate **current**, not voltage.

---

# 🏆 4️⃣ Professional-Grade IC Types That Fit Perfectly

These categories are ideal:

### 🔹 Texas Instruments

- TPS61165
    
- TPS61169
    
- TPS61042
    

### 🔹 Analog Devices / Linear

- LT3476
    
- LT3518
    

### 🔹 Monolithic Power Systems

- MP3429
    
- MP3202
    

All support:

- 2.7–5 V input
    
- 20V+ output
    
- PWM dimming
    
- External current sense resistor
    

---

# 🔌 5️⃣ How It Connects to Your STM32

Very clean:

STM32 PWM pin  ─────────► LED Driver PWM/DIM  
STM32 GPIO     ─────────► LED Driver EN (optional)  
Battery        ─────────► LED Driver VIN  
LED Driver OUT ─────────► Display Backlight

You run PWM at:

- ~1–20 kHz typically
    

Brightness control becomes:

__HAL_TIM_SET_COMPARE(&htimX, TIM_CHANNEL_Y, duty_cycle);

Real-time brightness control is trivial.

---

# 🎛 6️⃣ Dimming Strategy (Important Decision)

There are two types:

### 🔹 PWM Dimming (Recommended)

- LED fully on/off at high speed
    
- Preserves color temperature
    
- High efficiency
    
- Best for display backlights
    

### 🔹 Analog Current Dimming

- Adjusts LED current level
    
- Can shift color slightly
    
- Slightly lower efficiency
    

For a 1000-nit IPS panel:

👉 **Use PWM dimming.**

---

# 🔥 7️⃣ Why Constant Current Matters

Your datasheet says:

> The constant current driving method is suggested.

That’s because:

- LED forward voltage changes with temperature
    
- Battery voltage changes with charge
    
- Without current regulation → brightness unstable
    
- Risk of LED damage
    

A proper LED driver maintains:

I = constant 40 mA

No matter what.

---

# 🧱 8️⃣ Architecture Overview (Your Full System)

         LiPo Battery  
                │  
        Battery Protection  
                │  
        3.3V Regulator → STM32H750  
                │  
                ▼  
        Boost LED Driver  
                │  
        ~20V @ 40mA Constant Current  
                │  
          Display Backlight

PWM from STM32 controls brightness.

Very professional architecture.

---

# 🧠 9️⃣ What I Would Choose For You

Given:

- Battery powered
    
- 40 mA only
    
- Want small PCB
    
- Want real-time dimming
    

I would pick:

👉 A small SOT-23 or QFN boost LED driver with built-in switch.

Something like:

- TPS61165 class
    
- MP3202 class
    

These are literally designed for this use case (mobile LCD backlights).

---

# 🎯 10️⃣ Battery Consideration

Since you're LiPo powered:

You’ll also need:

- LiPo charge IC (USB-C charging likely)
    
- Protection circuit
    
- 3.3V regulator for MCU
    
- Possibly power path management
    

But that’s separate from backlight driver.

---

# ✅ Final Verdict

You are building this correctly.

With your specs:

- ✔ Yes, you need a boost constant current LED driver
    
- ✔ Yes, PWM dimming is ideal
    
- ✔ Yes, battery powered is totally viable
    
- ✔ Yes, STM32H750 can control brightness in real time easily
    

This is a clean, professional embedded power architecture



# Wiring 

- 1 PWM jumper wire
    
- 1 GND jumper wire
    
- 2 battery wires
    
- 2 LED wires