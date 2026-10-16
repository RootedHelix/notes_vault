Related to [[Embedded]], [[HSI High Speed Internal Oscillator]], [[PLL Phase Locked Loops]], [[HSE High Speed External Oscillator or Crystal]]
#embedded #pll_hsi_hse

Yes — exactly.

**HSI and HSE are clock sources whose job is to generate a stable oscillation.**

But they do it in different ways.

---

# 🧭 First: What They Actually Are

Inside an STM32 (like your F446RE), you have:

- **HSI** = High-Speed Internal oscillator
    
- **HSE** = High-Speed External oscillator
    

Both exist to create a repeating electrical waveform that the MCU uses as its timing reference.

---

# 🔵 HSI — Internal Oscillator

HSI is:

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

---

# 🟢 HSE — External Crystal Oscillator

HSE uses:

> A quartz crystal connected to the MCU pins.

Example crystal:

![Image](https://www.jameco.com/Jameco/Products/MakeImag/27887.jpg)

![Image](https://cdn-shop.adafruit.com/970x728/2213-00.jpg)

![Image](https://community.st.com/t5/image/serverpage/image-id/58984i505E1FA6A595E57F/image-size/large?px=999&v=v2)

![Image](https://i.sstatic.net/jMNiy.jpg)

Quartz has a property called:

**Piezoelectric effect**

When voltage is applied:

- The crystal physically vibrates.
    
- It vibrates at a very specific natural frequency.
    

That vibration becomes electrical oscillation.

HSE is:

- Much more accurate
    
- More stable over temperature
    
- Used when precise timing is required (USB, Ethernet, etc.)
    

---

# ⚙️ So Yes — They Create Oscillation

They generate a periodic waveform.

That waveform becomes:

SYSCLK (after possible PLL multiplication)

And that clock drives:

- CPU instruction timing
    
- AHB bus
    
- APB buses
    
- Timers
    
- Peripherals
    

---

# 🔁 Where PLL Fits In

The PLL (Phase-Locked Loop) does NOT create oscillation from nothing.

It takes:

HSI or HSE oscillation

Then multiplies it.

Example:

- HSE = 8 MHz
    
- PLL multiplies ×10
    
- SYSCLK = 80 MHz
    

Still oscillation.  
Just faster.

---

# 🧠 Big Picture

Power (3.3V DC) keeps the chip alive.

HSI or HSE creates rhythm.

PLL adjusts the tempo.

CPU moves forward one tick at a time.

---

If you'd like, next we can go deeper into:

- What physically makes an RC oscillator oscillate
    
- How the crystal’s mechanical vibration becomes voltage
    
- Or how the PLL multiplies frequency without losing sync
    

You’re now asking real silicon-level questions.