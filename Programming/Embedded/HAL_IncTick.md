Related to [[Embedded]], [[HAL Library]], [[SYSTick]]
#embedded #hal_inctick

```C

__weak void HAL_IncTick(void)  
{  
  uwTick += uwTickFreq;  
}

```

Allows for user overriding and 

#uwtick

---

## What Is `uwTick`?

`uwTick` is a **global volatile counter** inside the HAL.

In the STM32 HAL source (typically in `stm32f4xx_hal.c`) you’ll find:

```c
__IO uint32_t uwTick;
```

That means:

- `__IO` → volatile (can change in interrupts)
    
- `uint32_t` → 32-bit unsigned integer
    
- Global → accessible across HAL
    

---

### What Does It Represent?

`uwTick` = **number of milliseconds since system startup**

It is incremented every time SysTick fires.

So if your SysTick is configured for 1ms:

|Time|uwTick|
|---|---|
|Startup|0|
|1 ms|1|
|100 ms|100|
|1000 ms|1000|
|10 sec|10000|

It’s just a millisecond counter.

---

##  What Does `HAL_IncTick()` Do?

```c
uwTick += uwTickFreq;
```

By default:

```c
uwTickFreq = 1;
```

So it increments by 1 each SysTick interrupt.

So effectively:

```c
uwTick++;
```

---

## ⏱ Where Is This Called?

Inside:

```c
void SysTick_Handler(void)
{
    HAL_IncTick();
    HAL_SYSTICK_IRQHandler();
}
```

So:

1. SysTick fires every 1ms
    
2. HAL_IncTick() runs
    
3. uwTick increments
    
4. HAL time advances
    

---

## What Uses `uwTick`?

### 1️ HAL_Delay()

Internally:

```c
uint32_t tickstart = HAL_GetTick();
while((HAL_GetTick() - tickstart) < Delay)
{
}
```

`HAL_GetTick()` simply returns `uwTick`.

So `uwTick` is literally what powers delays.

---

### 2️ Timeouts in Drivers

I2C  
SPI  
UART  
Flash drivers

They all use `HAL_GetTick()` for timeouts.

If `uwTick` doesn’t increment:

Everything time-based breaks.

---

## What Is `uwTickFreq`?

This allows HAL to change tick resolution.

Possible values:

- `HAL_TICK_FREQ_DEFAULT` → 1ms
    
- `HAL_TICK_FREQ_100HZ` → 10ms
    
- etc.
    

So instead of incrementing by 1, it could increment by 10.

This allows lower interrupt frequency if desired.

---

## Why Is The Function `__weak`?

```c
__weak void HAL_IncTick(void)
```

`__weak` means:

> You are allowed to override this function.

If you define your own `HAL_IncTick()` in your project, the linker will use yours instead of HAL’s.

This is how:

- FreeRTOS replaces the tick handler
    
- Custom timers replace SysTick
    
- Low-power modes use alternate timing sources
    

---

## ⚡ Embedded Insight (Important)

`uwTick` will overflow.

Since it’s 32-bit:

```
Max value = 4,294,967,295 ms
```

That’s about:

```
~49.7 days
```

After ~49 days:

```
uwTick rolls back to 0
```

But the subtraction method:

```c
HAL_GetTick() - start
```

Still works correctly because unsigned overflow is well-defined in C.

That’s intentional embedded design.

---

## Conceptually

Think of `uwTick` as:

> The firmware’s stopwatch since boot.

SysTick interrupt → increments uwTick → everything time-based depends on it.

---

## If You Want To Verify In GDB

You can literally do:

```
(gdb) p uwTick
```

And watch it increase every millisecond (if interrupts are running).

---

## Summary

`HAL_IncTick()`:

- Runs every SysTick interrupt
    
- Increments global millisecond counter
    
- Powers all HAL timing
    
- Is overrideable
    
- Will overflow after ~49 days (safely)
    

---

You’re now at the level where you’re no longer “using HAL” — you’re understanding how it works.

If you want next:

- 🧠 How to implement your own tick timer without HAL
    
- 🛠 How to move SysTick to a hardware timer (TIM2, etc.)
    
- ⚙️ How FreeRTOS hijacks this mechanism
    

What direction do you want to go?