Related to [[Embedded]], [[SYSTick]]
#embedded #nvic #systick 

Excellent. Now you’re stepping into the real core of embedded systems.

If SysTick is the **alarm**, then the NVIC is the **traffic controller**.

Let’s build this cleanly.

---

## What Is the NVIC?

**NVIC** = **Nested Vectored Interrupt Controller**

It is part of the ARM Cortex-M core (not an STM32 peripheral).

On your STM32F446RE, it lives inside the Cortex-M4 CPU itself.

---

## What Does It Do?

The NVIC:

1. Receives interrupt requests
    
2. Decides which one has priority
    
3. Interrupts the CPU
    
4. Saves CPU state automatically
    
5. Jumps to the correct handler
    
6. Supports interrupt nesting (higher priority can interrupt lower)
    

It is the **interrupt traffic manager**.

---

## Big Picture Architecture

![[nvic_1.jpg]]
![[nvic_2.jpg]]
![[nvic_3.jpg]]
![[nvic_4.jpg]]
Flow looks like:

```
Peripheral (TIM2, USART, SysTick)
        ↓
Interrupt Request Line
        ↓
NVIC
        ↓
CPU jumps to handler
```

---

## Without NVIC

If the NVIC didn’t exist:

- The CPU would not know which interrupt fired.
    
- It couldn’t prioritize.
    
- It couldn’t preempt lower-priority work.
    
- Real-time systems would not exist.
    

---

## What “Nested” Means

Nested means:

A higher priority interrupt can interrupt a lower priority interrupt.

Example:

- USART interrupt running
    
- Emergency stop interrupt fires (higher priority)
    
- NVIC pauses USART handler
    
- Runs emergency handler
    
- Returns to USART
    

That’s real-time control.

---

## What “Vectored” Means

Vectored means:

Each interrupt has a fixed address in the **vector table**.

The CPU doesn’t check conditions manually.

It jumps directly to the correct handler.

Fast.  
Deterministic.

---

## How It Works Mechanically

When an interrupt fires:

1. CPU finishes current instruction
    
2. CPU pushes registers onto stack automatically
    
    - R0–R3
        
    - R12
        
    - LR
        
    - PC
        
    - xPSR
        
3. CPU loads handler address from vector table
    
4. Executes handler
    
5. On `return`, CPU restores registers automatically
    

All of this is done in hardware.

No compiler magic.

---

## Example: SysTick Case

For SysTick:

- SysTick counter hits zero
    
- Hardware raises exception
    
- NVIC sees SysTick interrupt
    
- NVIC checks priority
    
- CPU jumps to `SysTick_Handler()`
    

You didn’t write that routing logic.  
The NVIC handled it.

---

# ⚙ What You Can Configure in NVIC

You can:

- Enable/disable interrupts
    
- Set priority levels
    
- Set priority grouping
    
- Trigger software interrupts
    

Example:

```c
NVIC_SetPriority(TIM2_IRQn, 1);
NVIC_EnableIRQ(TIM2_IRQn);
```

That tells NVIC:

- TIM2 interrupt exists
    
- It has priority 1
    
- It is allowed to interrupt
    

---

## Why NVIC Is So Important Professionally

Everything in embedded systems is interrupt-driven:

- UART receive
    
- SPI transfer complete
    
- Timer overflow
    
- ADC conversion complete
    
- DMA finished
    
- External button press
    

All go through NVIC.

Understanding NVIC = understanding embedded systems.

---

## Mental Model Upgrade

Think of the MCU like this:

- Main loop = background worker
    
- NVIC = real-time supervisor
    
- Interrupts = urgent tasks
    

Main runs when nothing urgent is happening.

Interrupts preempt when something urgent occurs.

---

## Where You Are Now

You understand:

- SysTick generates interrupt
    
- NVIC routes interrupt
    
- CPU jumps to handler
    
- HAL calls your callback
    
- Your timing system updates
    

You are now operating at real firmware architecture level.

---

Next logical deep dive:

Do you want to see:

1. How the vector table in `startup_stm32f446xx.s` connects NVIC to handlers?
    
2. How interrupt priorities actually work numerically?
    
3. What happens on the stack during interrupt entry?
    
4. Or configure your first peripheral interrupt manually (TIM2)?
    

Pick your direction.