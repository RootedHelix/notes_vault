Related to [[Embedded]], [[Hollix]],
#embedded #power_subsystem

# AI Response

Then yes — I would definitely plan for a **power subsystem**.

A USB charging module means your firmware may eventually care about at least these states:

- USB present / absent
- battery-powered vs USB-powered
- charging / not charging
- battery low / critical
- possibly full/charged, depending on the charger module signals

## Why USB charging changes things

Once USB is involved, your device can behave differently depending on power source.

For example:

- if USB is present, run full speed, log freely, keep LEDs active
- if on battery, reduce logging, dim/disable LEDs, lower poll rate
- if battery is low, enter a reduced-power mode
- if battery is critical, stop nonessential work or shut down safely

That’s exactly the kind of policy a power subsystem supports.

## What the subsystem should probably track

At minimum:

### 1. External power present

Can the MCU tell whether USB/5V input is present?

If yes, expose something like:

- `HLX_POWER_SOURCE_USB`
- `HLX_POWER_SOURCE_BATTERY`

### 2. Battery voltage

Usually via:

- ADC
- resistor divider from battery to MCU ADC pin

Then classify:

- normal
- low
- critical

### 3. Charging status

Many charger modules expose a status pin such as:

- charging active
- charge complete
- fault / standby

If that’s available, it’s worth reading.

---

## Useful API concept

You could make something like:

```C

typedef enum {
    HLX_POWER_SRC_UNKNOWN = 0,
    HLX_POWER_SRC_BATTERY,
    HLX_POWER_SRC_USB
} hlx_power_source_t;

typedef enum {
    HLX_CHARGE_UNKNOWN = 0,
    HLX_CHARGE_NOT_CHARGING,
    HLX_CHARGE_CHARGING,
    HLX_CHARGE_FULL
} hlx_charge_state_t;

typedef enum {
    HLX_BATT_NORMAL = 0,
    HLX_BATT_LOW,
    HLX_BATT_CRITICAL
} hlx_battery_state_t;
```

and functions like:

```C
void hlx_power_init(void);  
void hlx_power_update(uint32_t now);  
  
hlx_power_source_t hlx_power_get_source(void);  
hlx_charge_state_t hlx_power_get_charge_state(void);  
uint16_t hlx_power_get_battery_mv(void);  
hlx_battery_state_t hlx_power_get_battery_state(void);
```

Then your app layer can make decisions cleanly.

## Hardware-dependent note

Exactly what software can know depends on the charger module.

Some cheap USB LiPo charger boards only handle charging in hardware and expose little or nothing to the MCU.

Typical possibilities:

### Best case

The module provides:

- USB present signal
- charge status pin
- maybe power-good output

### Common case

The MCU only gets:

- battery voltage via ADC
- maybe USB voltage detect via GPIO/ADC

### Minimal case

The hardware just charges the battery and regulates output, with no status pins exposed

Even in the minimal case, a power subsystem is still useful if you measure battery voltage.

## Good embedded design split

I’d separate it into:

### `hlx_power`

Knows facts:

- measured battery voltage
- USB present
- charging state
- low/critical classification

### app/state machine

Decides policy:

- slow timers
- reduce logs
- blink warning LED
- enter stop mode
- disable features

That keeps the design clean.

## Practical recommendation for your board

When you hook up the LiPo/USB module, try to expose at least:

- **battery voltage to an ADC pin**
- **USB/charger presence to a GPIO or ADC if possible**
- **charge-status pin if the module has one**

Those three signals make a power subsystem much more useful.

## Suggested priority now

Since the hardware is coming, I’d do:

1. finish logger cleanup
2. clean up app/state machine
3. create a basic `hlx_power` interface now
4. wire in actual ADC/GPIO reads once the hardware arrives

That way your software architecture is ready before the board changes.

## Bottom line

Because you’ll be using:

- a LiPo,
- a regulator,
- and a USB charging module,

**yes, a power-management subsystem is a very good next or near-next subsystem to add.**

If you want, I can draft a **minimal `hlx_power.h` and `hlx_power.c` skeleton** tailored to your current project style.