Related to [[Hollix]], [[Hollix Documentation]] 
#hollix #embedded #hollix_timeline

Here is the timeline and plan formatted by AI for this project:

# AI Response

A realistic path is:

```text
MVP optical demo:              2–4 weeks
Embedded display prototype:    8–12 weeks
Battery-powered Hollix unit:   12–18+ weeks
Portfolio-ready device:        16–24 weeks
```

That assumes you work steadily, avoid custom PCB overreach early, and define “complete” as:

```text
Battery-powered STM32 device
driving the 480×480 RGB display
with controllable high-brightness backlight
inside a simple Pepper’s ghost enclosure
running your Hollix firmware stack
```

The project should be completed in **phases**, not as one big wiring attempt.

---

# Recommended project path

## Phase 0 — Lock the prototype scope

**Goal:** define what version 1 must do.

For the first complete version, keep the feature set tight:

```text
Display bright animated object / UI
Reflect it on angled glass
Run from battery or USB
Control backlight brightness with PWM
Log system state over UART
Use your event/timer/watchdog/logger architecture
```

Do **not** start with:

```text
touch input
Wi-Fi
OTA updates
complex UI framework
custom STM32H750 PCB
speaker/audio
advanced sensors
```

Those are version 2 features.

Your first target should be:

> A single-purpose holographic display appliance.

---

# Phase 1 — Validate the optical concept

**Timeline:** 2–7 days  
**Risk:** low  
**Hardware needed:** display or temporary bright screen, glass/acrylic, black enclosure material

Before fighting the STM32 display interface, prove the Pepper’s ghost effect mechanically.

Use any bright screen temporarily:

```text
phone
tablet
laptop screen
existing monitor
the 4-inch display once backlight works
```

Test:

```text
glass angle
distance from display
image size
brightness needed
ambient light sensitivity
viewing angle
black background quality
```

Deliverable:

```text
A visible floating/reflected image in a darkened enclosure.
```

This matters because the project’s core value is optical. The embedded system supports the illusion; it is not the illusion by itself.

---

# Phase 2 — Bring up firmware on NUCLEO-H743ZI2

**Timeline:** 1–2 weeks  
**Risk:** low to medium  
**Hardware needed:** NUCLEO-H743ZI2, USB, serial monitor

Use the Nucleo alone first.

Bring up:

```text
hlx_sys_tick
hlx_timers
hlx_event
hlx_ringbuffer
hlx_logging
hlx_watchdog
hlx_app
PWM output / hlx_pwm-style backlight-control path
```

Minimum firmware milestones:

```text
UART logging works
event queue works
periodic timer events work
watchdog timeout events work
PWM output works
PWM duty cycle can be changed at runtime
button or debug GPIO event works
```

Suggested demo behavior:

```text
[BOOT] Hollix firmware starting
[INFO] timers initialized
[INFO] event queue initialized
[INFO] PWM backlight channel initialized
[INFO] system heartbeat
```

This gives you a stable firmware base before display complexity enters the system.

---

# Phase 3 — Backlight driver bring-up

**Timeline:** 1 week  
**Risk:** medium  
**Hardware needed:** TPS61176EVM-566, display backlight pins, current-limited supply, PWM from Nucleo

This should be done before RGB display bring-up.

Test only:

```text
LEDA
LEDK
backlight current
PWM dimming
thermal behavior
```

Do not connect the RGB bus yet.

Backlight test wiring concept:

```text
Power source / SYS rail → TPS61176EVM VIN
TPS61176EVM GND        → common GND
TPS61176EVM VOUT       → display LEDA
TPS61176EVM IFB sink   → display LEDK
STM32 PWM GPIO         → TPS61176EVM EN/PWM
```

Deliverable:

```text
Display backlight turns on
brightness is PWM controllable
current draw is measured
no overheating
```

This is a major milestone because the 1000-nit backlight is central to the hologram quality.

---

# Phase 4 — Build the correct power tree

**Timeline:** 1–2 weeks  
**Risk:** medium  
**Hardware needed:** MCP73871 Click, LiPo, 3.3 V buck-boost regulator, capacitors, switch, fuse/PTC

Your correct architecture is:

```text
USB / 5 V input
      ↓
MCP73871 Click
      ↓
SYS / OUT rail
      ├── 3.3 V buck-boost → STM32 + display logic
      └── TPS61176 boost driver → display backlight
```

Important correction:

```text
SparkFun AP63203 buck ≠ final LiPo regulator
```

It can work from 5 V input, but for a 1S LiPo battery system you want:

```text
1S LiPo/SYS → 3.3 V buck-boost
```

Deliverables:

```text
Device can run from USB input
device can run from LiPo
3.3 V rail remains stable
backlight driver does not brown out the system
grounds are common
current draw is measured
```

Do this before full display bring-up so you know the system rails are stable.

---

# Phase 5 — Display interface hardware

**Timeline:** 1–3 weeks depending on adapter/custom PCB  
**Risk:** high  
**Hardware needed:** 45-pin 0.5 mm FPC adapter or custom interposer PCB

This is the phase where you should be careful.

The display has a parallel RGB interface. That means you need many signals:

```text
RGB data lines
PCLK
DE
HSYNC
VSYNC
RESET
SPI/control pins
VCI
IOVCC
GND
LEDA
LEDK
```

Do not wire this with a loose breadboard bundle.

You need one of these:

```text
45-pin 0.5 mm FPC breakout
or
custom interposer PCB
```

Recommended interposer features:

```text
45-pin FPC connector
series resistors on high-speed RGB/PCLK lines
local decoupling capacitors
test pads
separate backlight connector
ground pins grouped cleanly
short path to Nucleo headers
```

Deliverable:

```text
Display physically connects to Nucleo safely and repeatably.
```

This is probably the biggest hardware gate in the whole project.

---

# Phase 6 — LTDC display bring-up

**Timeline:** 2–4 weeks  
**Risk:** high  
**Hardware needed:** NUCLEO-H743ZI2, display adapter, stable 3.3 V rail, backlight driver

Start with the simplest possible display output.

Do not start with animations. Start with test colors:

```text
full red screen
full green screen
full blue screen
white screen
black screen
checkerboard
vertical bars
```

Use:

```text
RGB565 first
single framebuffer
static image
no complex rendering
```

A 480×480 RGB565 framebuffer is:

```text
480 × 480 × 2 = 460,800 bytes
```

That is reasonable on STM32H743-class hardware, but you still need to be disciplined with memory.

Minimum display milestones:

```text
LTDC clock configured
GPIO alternate functions configured
timing values configured
framebuffer allocated
backlight enabled
PWM backlight brightness control verified
panel reset sequence works
panel init sequence works
solid color appears
test pattern appears
```

Deliverable:

```text
The STM32 drives the display directly.
```

This is the hardest technical milestone.

---

# Phase 7 — Hollix graphics demo

**Timeline:** 1–3 weeks  
**Risk:** medium  
**Hardware needed:** working display

Now make it look like Hollix.

Start with graphics that are favorable for Pepper’s ghost:

```text
black background
bright cyan/white object
high contrast
simple animation
large clean shapes
minimal background clutter
```

Good first visuals:

```text
floating cube
rotating wireframe object
holographic logo
pulsing circular UI
small character/sprite silhouette
terminal-style boot animation
```

Avoid low-contrast art at first. Pepper’s ghost works best when the source image is bright and the background is black.

Firmware integration:

```text
timer event → animation tick
event queue → UI/action state changes
watchdog → system health
logger → boot/debug state
PWM → brightness control
```

Deliverable:

```text
A custom Hollix animation runs on the embedded display and reflects cleanly on glass.
```

---

# Phase 8 — Enclosure and optical frame

**Timeline:** 2–4 weeks  
**Risk:** medium  
**Hardware needed:** acrylic/glass, enclosure material, mounts, fasteners

This should happen after you know the display/backlight works, but you can prototype the shape earlier.

Core enclosure requirements:

```text
display fixed in position
glass fixed at consistent angle
interior matte black
ambient light blocked
wiring strain-relieved
battery secured
USB charging port accessible
power switch accessible
debug port accessible
```

The enclosure matters a lot. A bad enclosure will make a working display look weak.

Deliverable:

```text
Self-contained tabletop Hollix prototype.
```

---

# Phase 9 — Battery integration and runtime testing

**Timeline:** 1–2 weeks  
**Risk:** medium  
**Hardware needed:** LiPo, MCP73871, buck-boost, current measurement

Measure current in real operating modes:

```text
boot
idle display
bright display
max backlight
animation running
charging while running
```

You want to know:

```text
average current draw
peak current draw
backlight current draw
3.3 V rail current draw
battery runtime
thermal behavior
charging behavior
```

Approximate runtime formula:

```text
Runtime hours ≈ battery capacity mAh / average current mA
```

Example:

```text
4000 mAh / 800 mA ≈ 5 hours
4000 mAh / 1200 mA ≈ 3.3 hours
4000 mAh / 2000 mA ≈ 2 hours
```

The real value will be lower because of converter losses.

Deliverable:

```text
Known runtime and stable battery-powered operation.
```

---

# Phase 10 — Polish, documentation, and portfolio packaging

**Timeline:** 2–4 weeks  
**Risk:** low  
**Hardware needed:** completed prototype

This is where you turn it from “project that works” into “project that sells your skill.”

Document:

```text
system block diagram
power tree
firmware architecture
event system
logging/DMA design
watchdog/timer system
PWM backlight-control design
display bring-up notes
optical design
known limitations
version 2 roadmap
```

Portfolio deliverables:

```text
clean GitHub README
short demo video
wiring diagram
firmware architecture diagram
photos of prototype
technical write-up
lessons learned
```

This project is strong for an embedded resume only if the documentation makes the complexity visible.

---

# Overall timeline

## Aggressive timeline: 8–12 weeks

This assumes minimal enclosure polish and no major display issues.

|Phase|Time|
|---|--:|
|Optical proof|2–4 days|
|Nucleo firmware bring-up|1 week|
|Backlight bring-up|1 week|
|Power tree|1 week|
|Display adapter/interposer|1–2 weeks|
|LTDC display bring-up|2–3 weeks|
|Graphics demo|1 week|
|Basic enclosure|1–2 weeks|
|Documentation|1 week|

## Realistic timeline: 12–18 weeks

This is the better estimate.

|Phase|Time|
|---|--:|
|Optical proof|1 week|
|Firmware base|1–2 weeks|
|Power/backlight|2–3 weeks|
|Display interconnect|2–4 weeks|
|LTDC/display bring-up|3–5 weeks|
|Graphics/animation|2–3 weeks|
|Enclosure|2–4 weeks|
|Battery/runtime|1–2 weeks|
|Portfolio polish|2 weeks|

## Professional-quality timeline: 4–6 months

This includes cleaner enclosure, better PCB/interposer, runtime validation, and strong documentation.

---

# Critical path

The critical path is:

```text
FPC/interposer hardware
        ↓
LTDC display bring-up
        ↓
backlight brightness control
        ↓
optical enclosure
        ↓
battery integration
```

Your firmware modules are important, but they are not the biggest schedule risk. The biggest risks are physical/electrical:

```text
display connector
RGB signal routing
panel init sequence
LTDC timing
backlight current configuration
battery regulator choice
mechanical optical alignment
```

---

# What I would do first, in order

## Step 1

Order or design the display adapter/interposer.

Without this, the real display bring-up cannot proceed cleanly.

## Step 2

Order a proper 1S LiPo to 3.3 V buck-boost regulator.

Do not build the battery-powered version around the AP63203.

## Step 3

Bring up the Nucleo firmware stack without the display.

Get logging, timers, events, watchdog, and PWM working.

## Step 4

Test the display backlight with the TPS61176EVM.

Do this before attempting RGB.

## Step 5

Build a temporary optical mockup.

Use any bright screen first. Confirm that the physical illusion works.

## Step 6

Connect the RGB display through the adapter/interposer.

Start with solid color LTDC output.

## Step 7

Build the first Hollix visual demo.

Keep it simple, bright, and high-contrast.

## Step 8

Move from bench power to battery power.

Validate runtime and charging behavior.

## Step 9

Package into enclosure.

Make it physically stable and visually convincing.

## Step 10

Document it like an engineering project.

That is what makes it portfolio-grade.

---

# Suggested version milestones

## Hollix V0 — Optical proof

```text
Any screen
angled glass
floating image visible
no embedded system required
```

This proves the core idea.

## Hollix V1 — Embedded bench prototype

```text
Nucleo-H743ZI2
RGB display working
backlight driver working
PWM backlight dimming working
bench power or USB power
simple animation
UART logging
```

This proves the embedded system.

## Hollix V2 — Battery prototype

```text
LiPo
MCP73871 charging/power path
3.3 V buck-boost
backlight boost driver
PWM brightness control
runtime measured
basic enclosure
```

This proves portability.

## Hollix V3 — Portfolio prototype

```text
clean enclosure
documented firmware architecture
demo video
GitHub README
system diagrams
known limitations
future roadmap
```

This proves engineering maturity.

## Hollix V4 — Product-style revision

```text
custom PCB
STM32H750 target
external flash/SDRAM if needed
USB-C charging
mechanical refinement
more graphics/features
```

This is the version to attempt only after V1/V2 work.

---

# My recommended immediate next actions

Do these next:

```text
1. Find/order a 45-pin 0.5 mm FPC breakout or start a small interposer PCB.
2. Pick a proper 1S LiPo → 3.3 V buck-boost regulator.
3. Bring up PWM on the NUCLEO-H743ZI2.
4. Test TPS61176EVM with the display backlight only.
5. Build a crude Pepper’s ghost mockup with black foam board/acrylic/glass.
```

The project becomes much easier once you split it into:

```text
optics
firmware
display interface
power
mechanical
documentation
```

The correct strategy is not “wire everything together.” The correct strategy is **prove each subsystem independently, then integrate one boundary at a time.**