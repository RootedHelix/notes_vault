Related to [[Embedded]], [[System Clock Tree]], [[STM32 Week1]] 
#embedded #systick

## ⏱ What Is SysTick?

### 🧠 Concept First

**SysTick** is a built-in 24-bit timer inside the ARM Cortex-M core.

It is:

- Part of the Processor Core (not a peripheral like TIM2)
    
- Present on all Cortex-M chips
    
- Designed to generate periodic interrupts
    

On your **STM32F446RE**, it lives inside the Cortex-M4 core.

Think of it as:

> The MCU’s heartbeat timer.

---

## 🏗 Where It Lives

![[systick_1.jpg]]

![[systick_2.jpg]]
![[sys_tick_3.jpg]]
![[systick_4.jpg]]
SysTick:

- Is a 24-bit down counter
    
- Counts down from a value you set
    
- When it hits 0 → triggers an interrupt
    
- Reloads automatically
    
- Repeats
    

---

## Why SysTick Exists

It is primarily used for:

- OS scheduling (FreeRTOS tick)
    
- Millisecond time base
    
- Delays
    
- Periodic tasks
    

CubeMX actually uses SysTick for `HAL_Delay()`.

But we’re going to go deeper than HAL.

---

## How It Works (Mechanically)

SysTick registers (core registers):

|Register|Purpose|
|---|---|
|`SYST_CSR`|Control & status|
|`SYST_RVR`|Reload value|
|`SYST_CVR`|Current value|
|`SYST_CALIB`|Calibration|

Operation:

1. You set reload value (RVR)
    
2. Counter loads that value
    
3. It counts down every clock cycle
    
4. When it reaches 0:
    
    - COUNTFLAG is set
        
    - Interrupt fires (if enabled)
        
    - Reload happens automatically
        

---

## Example Goal: 1ms System Tick

If your system clock is 16 MHz:

16,000,000 Hz  
= 16,000 cycles per millisecond

So you set:

`SysTick->LOAD = 16000 - 1;`

Now SysTick fires every 1ms.

---

## Minimal Bare-Metal Implementation

This is what professional firmware looks like:

```C
#include "stm32f446xx.h"  

volatile uint32_t msTicks = 0;  

void SysTick_Handler(void) {     
	msTicks++;   // runs every 1ms 
}  
void systick_init(void) {     
	SysTick->LOAD  = 16000 - 1;              // 1ms at 16MHz     
	SysTick->VAL   = 0;                      // clear current value     
	SysTick->CTRL  = (1 << 0) |              // ENABLE   
					 (1 << 1) |              // TICKINT   
					 (1 << 2);               // CLKSOURCE (AHB) 
}  
int main(void) {    
	systick_init();    
	while (1)     {      
		if (msTicks >= 1000)      
		{     
			msTicks = 0;             // toggle LED here
		}     
	} 
}
```


---

## What Just Happened?

You:

- Configured a hardware timer
    
- Enabled an interrupt
    
- Created a system time base
    
- Built a non-blocking timing mechanism

HAL sets up SysTick and calls a weak callback. I provide my own callback to hook into the interrupt chain. The `msTicks` variable must be volatile because it is modified in interrupt context and read in main, and the compiler must not optimize based on the assumption that it never changes.

---

## Professional Insight

Embedded professionals use SysTick for:

#### 1️ Cooperative scheduling

`if (task_timer >= 10) { run_task(); }`

#### 2️ State machines with time
```C
switch(state) {
case WAIT:
if (msTicks - start >= 50)
state = NEXT;
}
```
#### 3️ RTOS tick source

FreeRTOS uses SysTick as the scheduler interrupt.

---

## ⚠ Important Concepts You Must Understand

### 1️ SysTick runs at core clock speed

If you change system clock → timing changes.

### 2️ It is only 24-bit

Max reload:

`2^24 - 1 = 16,777,215`

At 16 MHz:

`16,777,215 / 16,000,000 ≈ 1.048 seconds`

So max interval ≈ 1 second.

For longer timing → use hardware timers.

---

## SysTick vs TIM2

|SysTick|TIM2|
|---|---|
|Core timer|Peripheral timer|
|24-bit|32-bit|
|Meant for system tick|Meant for precise timing|
|Limited flexibility|Much more configurable|

---

## Where This Fits In Your Growth

Right now you are learning:

- Clock system
    
- Registers
    
- Interrupts
    
- Timing
    
- Deterministic control
    

This is **core embedded systems engineering**.

Not Arduino.  
Not scripting.  
Real firmware.

# What is the Order of Execution From SYStick to my function

Yes — but let’s make it crystal clear at the hardware level.

The callback does **not** interact directly with SysTick hardware.

It interacts with HAL’s interrupt chain.

Here’s the precise flow on your STM32F446RE:

---

##  Full Interrupt Chain

### 1️ SysTick hardware counts down

Built into the Cortex-M4 core.

### 2️ Counter hits zero

Hardware raises a **SysTick exception**.

### 3️ NVIC handles it
[[NVIC]]
The Nested Vectored Interrupt Controller (NVIC) routes the exception.

### 4️ CPU jumps to `SysTick_Handler()`

That function lives in:

```
stm32f4xx_it.c
```

And inside it you’ll see:

```c
void SysTick_Handler(void)
{
  HAL_IncTick();
  HAL_SYSTICK_IRQHandler();
}
```

### 5️ HAL runs its handler

Inside HAL:

```c
void HAL_SYSTICK_IRQHandler(void)
{
  HAL_SYSTICK_Callback();
}
```

### 6️ Your callback runs

Because HAL defines:

```c
__weak void HAL_SYSTICK_Callback(void)
{
}
```

And you provided your own version:

```c
void HAL_SYSTICK_Callback(void)
{
    msTicks++;
}
```

Your function replaces the weak one.

---

#  So What Is The Callback?

The callback is:

> A user extension point that HAL calls during the SysTick interrupt.

It does not configure SysTick.  
It does not trigger SysTick.  
It does not interact with NVIC directly.

It simply runs inside the interrupt context.

---

## The Clean Mental Model

```
SysTick hardware
    ↓
NVIC
    ↓
SysTick_Handler()
    ↓
HAL internal tick update
    ↓
HAL_SYSTICK_Callback()
    ↓
Your msTicks++
```

You are attaching logic to the interrupt chain.

---

## Why HAL Designed It This Way

Because:

- Users often need periodic 1ms tasks.
    
- HAL already owns SysTick.
    
- Rather than forcing users to rewrite the handler,  
    HAL exposes a weak callback hook.
    

This keeps architecture clean.

---

## Professional Takeaway

You now understand:

- Hardware interrupt
    
- NVIC routing
    
- Vector table jump
    
- HAL handler
    
- Weak callback override
    
- Shared volatile state
    
- Cooperative time usage
    

That is solid interrupt literacy.

---

If you want to go one level deeper:

We can open `startup_stm32f446xx.s` and see exactly how the vector table maps SysTick to the handler.

That’s where embedded systems stops feeling magical and starts feeling mechanical.



