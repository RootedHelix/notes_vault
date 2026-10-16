Related to [[Hollix]], [[Connecting The STM32H750 to the HighBrightness Display Module]]
#hollix #embedded #hollix_wiring_architecture

## AI Response

Below is a **typical safe wiring architecture** for using:

- STM32H750XBH6
    
- MCP73871 (via MCP73871 Click board)
    
- TPS61176EVM-566
    

This configuration lets the system:

- run from **USB power**
    
- **charge the battery**
    
- automatically **switch to battery when USB is removed**
    
- drive **LEDs using the boost driver**
    

---

# System Power Architecture

![Image](https://eu.mouser.com/images/marketingid/2017/microsites/101396856/MCP73871_AC.png)

![Image](https://www.edomtech.com/upload/article_m/ALL_article_23A03_HTCyInznOK.png)

![Image](https://cdn.prod.website-files.com/66f7e926752b1858a611ee9a/68c012417fe1a22a10eca1bc_685c6e9f9e85ce5bff33eeb1_Sometimes-Datasheets-Lead-You-Wrong-With-Lithium-Chargers-1024x623.png)

![Image](https://cdn.prod.website-files.com/66f7e926752b1858a611ee9a/685c6e9f9e85ce5bff33eeba_Lithium-Ion-Charging-Circuit-That-Passes-Voltage-Through-Load-When-Charging-1-1024x430.png)

Conceptually the wiring looks like this:
```

USB / External 5V
        │
        ▼
   MCP73871 Charger
      │       │
      │       └── LiPo Battery
      │
      └── SYS (system power rail)
              │
              ├── 3.3V regulator → MCU
              │        │
              │        └── STM32H750XBH6
              │
              └── LED driver / peripherals
                       │
                       └── TPS61176EVM-566```

The **SYS output of the charger powers the whole system**.

This is important because the MCP73871 includes **load sharing**, meaning:

- USB present → system powered by USB
    
- USB removed → system powered by battery automatically
    

---

# Step-by-Step Wiring

## 1. USB Power → Charger

```
5V USB
  │
  ├── VBUS → MCP73871 VIN
  └── GND  → MCP73871 GND
```

Wire recommendation:

- **22–24 AWG**
    

---

# 2. Battery → Charger

```
LiPo +
  │
  └── MCP73871 VBAT

LiPo -
  │
  └── GND
```

Wire recommendation:

- **22 AWG**
    

Keep these wires short.

---

# 3. Charger → System Power Rail

The key pin is **SYS**.

```
MCP73871 SYS
      │
      ├── 3.3V regulator → STM32
      │
      └── VIN → TPS61176 LED driver
```

Important:

The STM32H750 requires **3.3 V**, so you must use a **voltage regulator** if SYS is higher.

Typical chain:

```
SYS (3.7–5V)
   │
   ▼
3.3V LDO / buck regulator
   │
   ▼
STM32H750 VDD
```

---

# 4. MCU Connections

The MCU needs:

```
3.3V → VDD pins
GND → GND pins
```

Optional control:

```
STM32 GPIO → LED driver enable
STM32 ADC → battery voltage sense
```

Signal wires:

- **24–26 AWG**
    

---

# 5. LED Driver Wiring

```
SYS / VIN
   │
   └── TPS61176 VIN

GND
   │
   └── TPS61176 GND
```

LED output:

```
TPS61176 LED+
TPS61176 LED-
      │
      └── LED string
```

Wire recommendation:

|Connection|Wire|
|---|---|
|VIN|22–24 AWG|
|LED output|22 AWG|

Boost converters produce switching noise, so **short wires are best**.

---

# Grounding Layout (Important)

All grounds should connect together:

```
Battery GND
     │
USB GND
     │
MCP73871 GND
     │
STM32 GND
     │
LED Driver GND
```

Use a **star grounding layout** if possible.

---

# Optional (Recommended) Protection

Good designs include:

### Input protection

```
USB → fuse → charger
```

### Bulk capacitors

Near the LED driver:

```
10 µF
22 µF
```

Near the MCU regulator:

```
10 µF + 0.1 µF
```

---

# Practical Prototype Layout

For a bench prototype:

```
USB → MCP73871 Click
         │
         ├── Battery (JST cable)
         │
         └── SYS
              │
              ├── 3.3V regulator → STM32 board
              │
              └── TPS61176 EVM → LEDs
```

---

# Common Mistakes to Avoid

1. **Powering STM32 directly from LiPo**
    

LiPo voltage:

```
4.2 V full
3.0 V empty
```

The STM32H750 requires **3.3 V regulated**.

---

2. **Running LED current through thin jumper wires**
    

Use **22 AWG** for LED current paths.

---

3. **Long wires on switching regulator**
    

Boost converters become unstable with long leads.

---

# If Your Goal Is Embedded Firmware Work

A very typical system architecture is:

```
Battery
   │
Charger / power path
   │
3.3V regulator
   │
STM32H750
   │
SPI / PWM
   │
LED driver
```

This is a **real embedded hardware architecture used in portable devices**.

---

If you'd like, I can also show you **a clean breadboard/prototype layout for this exact system**, because placing the charger, regulator, MCU, and LED driver incorrectly can easily introduce noise or brownouts.

```


## Power Architecture 

The battery sits **between the charger IC and the system load**, with the charger managing both **charging and load sharing**. The **MCP73871** is designed specifically for this type of architecture.

---

# Power Chain Overview

The typical topology looks like this:

```
USB / External 5V
        │
        ▼
   MCP73871 Charger
      │       │
      │       └── LiPo Battery
      │
      └── SYS (system power rail)
              │
              ├── 3.3V regulator → MCU
              │        │
              │        └── STM32H750XBH6
              │
              └── LED driver / peripherals
                       │
                       └── TPS61176EVM-566
```

The key node is the **SYS pin**.

---

# Role of Each Power Node

## 1. USB Input

```
USB 5V → MCP73871 VIN
```

When USB power is available:

- The system load is powered from **USB**
    
- The battery is **charged simultaneously**
    

---

## 2. Battery Connection

The battery connects directly to the charger IC.

```
LiPo + → VBAT
LiPo - → GND
```

The charger manages:

- charging current
    
- termination
    
- thermal protection
    

The battery is **not directly powering the MCU**.

---

## 3. SYS Output (System Rail)

The **SYS pin** powers the entire device.

```
SYS → system electronics
```

This node automatically switches source:

|Condition|SYS powered by|
|---|---|
|USB present|USB input|
|USB removed|Battery|

This behavior is called **power-path management**.

---

# Why the Battery Is Not Wired Directly to the MCU

A Li-ion battery voltage varies significantly:

```
4.2 V  (fully charged)
3.7 V  (nominal)
3.0 V  (near empty)
```

The **STM32H750XBH6** requires a regulated **3.3 V rail**, so a regulator must be inserted:

```
SYS
 │
 ▼
3.3V regulator
 │
 ▼
STM32 VDD
```

---

# What Happens in Different Scenarios

## Case 1 — USB plugged in

```
USB → system load
USB → charger → battery
```

System runs from USB while battery charges.

---

## Case 2 — USB removed

```
Battery → SYS → system load
```

The device automatically switches to battery power.

---

## Case 3 — Heavy load

The MCP73871 dynamically splits current between:

- system load
    
- battery charging
    

This prevents the system from **browning out during charge**.

---

# Where the LED Driver Fits

The **TPS61176EVM-566** typically runs from the **SYS rail**:

```
SYS → LED driver VIN
```

Boost LED drivers are designed to tolerate a **wide input range**, which matches the SYS voltage variation.

---

# Practical Wiring Example

```
USB 5V
  │
  ▼
MCP73871
  │        │
  │        └── LiPo Battery
  │
  └── SYS
        │
        ├── 3.3V regulator → STM32H750
        │
        └── TPS61176 LED driver
```

---

# Key Design Principle

In portable embedded systems the battery **never directly powers the digital electronics**. Instead the chain is:

```
Battery
   │
Charger / power-path IC
   │
System rail
   │
Voltage regulator
   │
MCU and logic
```

This ensures:

- stable voltage
    
- safe battery charging
    
- automatic source switching
    

---

If you want, I can also show **a minimal real schematic for powering an STM32H750 from a LiPo battery**, which makes the charger–regulator–MCU relationship much clearer.