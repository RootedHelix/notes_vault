Related to [[Hollix]], [[STM32]]
#hollix #embedded #connecting_stm_to_display


## AI Response

Yes—conceptually correct, but there is an important clarification.

You **do not plug the display into a single pin labeled “RGB.”** Instead, the display connects to **many individual pins that together form the RGB interface** driven by the LTDC peripheral on the **STM32H750**.

When documentation or schematics say **“RGB interface”**, they mean a **parallel bus made up of multiple signals**.

---

# What “RGB Interface” Actually Means

Instead of one connector labeled RGB, the MCU provides **many pins like this**:

### RGB Data Lines

```
LTDC_R0
LTDC_R1
...
LTDC_R7

LTDC_G0
LTDC_G1
...
LTDC_G7

LTDC_B0
LTDC_B1
...
LTDC_B7
```

These pins carry the **color data for each pixel**.

---

### Display Timing Signals

These tell the display when each pixel and frame starts.

```
LTDC_CLK   (pixel clock)
LTDC_HSYNC
LTDC_VSYNC
LTDC_DE    (data enable)
```

---

# How the Physical Connection Works

The display module usually has a **40-pin FPC ribbon connector**.

That connector contains pins like:

|LCD Pin|Connects To|
|---|---|
|R0|LTDC_R0|
|R1|LTDC_R1|
|G0|LTDC_G0|
|B0|LTDC_B0|
|HSYNC|LTDC_HSYNC|
|VSYNC|LTDC_VSYNC|
|PCLK|LTDC_CLK|
|DE|LTDC_DE|

So electrically it becomes:

```
STM32H750 LTDC pins
        │
        │ (20–28 traces)
        ▼
40-pin LCD connector
        │
        ▼
ST7701S display controller
```

---

# What You Would Actually Do on Hardware

If you are building a board:

1. Place a **40-pin FPC connector** for the display.
    
2. Route the LTDC pins from the STM32 to the connector.
    

Example mapping:

```
STM32 Pin        LCD Connector Pin
----------------------------------
LTDC_R0  ------> R0
LTDC_R1  ------> R1
LTDC_G0  ------> G0
LTDC_B0  ------> B0
LTDC_HSYNC ---> HSYNC
LTDC_VSYNC ---> VSYNC
LTDC_CLK  ---> PCLK
LTDC_DE   ---> DE
```

Plus additional connections:

```
SPI → ST7701S initialization
GPIO → RESET
PWM/GPIO → Backlight
```

---

# Important Practical Point

If you are using a **development board**, you typically **do not wire the LTDC pins manually**.

Instead you plug the display into:

```
LCD connector
RGB display header
```

because the board already routes those LTDC signals internally.

---

✔ **Summary**

When documentation says the display uses an **RGB interface**, it means:

- **Many MCU pins together form the RGB bus**
    
- These connect to the **LCD ribbon connector**
    
- The LTDC hardware streams pixel data through those pins.
    

---

If you'd like, I can also show you **the actual LTDC pin cluster on the STM32H750 package diagram**, which makes it much easier to see **exactly where on the chip the display signals come from.**