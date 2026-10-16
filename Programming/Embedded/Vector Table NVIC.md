Related to [[Embedded]], [[Vector Table NVIC]]
#embedded #vector_table #nvic

Excellent — this is where embedded stops feeling abstract and starts feeling mechanical.

The **vector table** is the lookup table that tells the CPU:

> “When interrupt X happens, jump to function Y.”

It is the bridge between hardware interrupts and your C functions.

---

## What Is the Vector Table?

On ARM Cortex-M (like the one inside your STM32F446RE):

The vector table is:

- A table of memory addresses
    
- Located at the start of Flash (usually 0x08000000)
    
- Contains pointers to interrupt handlers
    

It is literally an array of function addresses.

---

## What It Looks Like Conceptually

It looks like this in memory:

```
Address        Content
--------------------------------
0x08000000 → Initial Stack Pointer
0x08000004 → Reset_Handler
0x08000008 → NMI_Handler
0x0800000C → HardFault_Handler
...
0x0800003C → SysTick_Handler
...
```

Each entry is 4 bytes (a 32-bit address).

When an interrupt fires:

1. CPU determines the interrupt number.
    
2. CPU multiplies that number by 4.
    
3. CPU adds it to vector table base address.
    
4. CPU loads the function pointer.
    
5. CPU jumps to it.
    

No searching.  
No conditionals.  
Direct jump.

That’s what “vectored” means.

---

## Example: SysTick

When SysTick fires:

- It has a fixed interrupt number.
    
- CPU calculates its vector table offset.
    
- CPU loads the address of `SysTick_Handler`.
    
- Execution jumps there immediately.
    

That’s how your callback eventually runs.

---

## Where It Lives in Your Project

In your project directory you have:

```
startup_stm32f446xx.s
```

That file defines the vector table in assembly.

Inside it you’ll find something like:

```asm
.section .isr_vector
.word _estack
.word Reset_Handler
.word NMI_Handler
.word HardFault_Handler
...
.word SysTick_Handler
```

Those `.word` entries are the vector table.

---

## Why This Is So Important

Without the vector table:

- The CPU would not know where to jump.
    
- Interrupts would not work.
    
- Reset would not work.
    
- HardFault handling would not work.
    

The vector table is the CPU’s interrupt map.

---

## What Happens at Boot

When your STM32 powers up:

1. CPU reads first word of Flash → sets stack pointer.
    
2. CPU reads second word → jumps to `Reset_Handler`.
    
3. `Reset_Handler` eventually calls `main()`.
    

All of that is driven by the vector table.

---

## Why It’s Called “Vector”

Because each interrupt has a fixed “vector” (direction) to its handler.

It’s literally a table of jump vectors.

---

## Big Mental Model

```
Hardware Event
     ↓
Interrupt number generated
     ↓
CPU looks up vector table
     ↓
Loads handler address
     ↓
Jumps to handler
```

Zero OS involvement.  
Pure hardware.

---

You are now at the point where:

- You understand SysTick
    
- You understand NVIC
    
- You understand vector tables
    
- You understand interrupt flow
    

This is real embedded systems literacy.

---

If you want next, we can:

- Open your actual `startup_stm32f446xx.s` and trace SysTick’s entry
    
- Or step through interrupt entry at the CPU stack level
    
- Or explain how bootloaders relocate the vector table
    

Where do you want to go next?