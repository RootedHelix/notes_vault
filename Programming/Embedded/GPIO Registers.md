Related to [[Embedded]], [[STM32 Week1]]
#embedded #stm32 #gpio_registers

## Main GPIO Registers

[[GPIO MODER]]
[[GPIO OTYPER]]
[[GPIO OSPEEDR]]
[[GPIO PUPDR]]
[[GPIO IDR]]
[[GPIO ODR]]
[[GPIO BSRR]]


# What Are GPIO Registers?

**GPIO registers are memory-mapped control registers inside the MCU that configure and control General Purpose Input/Output pins.**

Instead of calling a HAL function like:

`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);`

you can directly manipulate registers like:

`GPIOA->ODR |= (1 << 5);`

That’s what’s happening underneath the HAL.

---

## On Your Board: STM32F446RE

Each GPIO port (A, B, C…) has a block of registers mapped into memory.

Example:

- GPIOA base address → `0x40020000`
    
- GPIOB → `0x40020400`
    
- etc.
    

When you write to those addresses, you're directly controlling hardware.

## How GPIO Registers Fit Into the Bigger Picture

When you configure a pin:

1. RCC enables GPIO clock
    
2. MODER sets pin mode
    
3. OTYPER sets driver type
    
4. OSPEEDR sets speed
    
5. PUPDR sets pull configuration
    
6. IDR/ODR/BSRR read or write actual pin
    

That entire chain is hardware-level control.