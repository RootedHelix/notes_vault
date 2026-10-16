# Embedded Systems

This note serves as the **hub for all embedded systems concepts, architecture, and development workflows** in this vault.

---

# Microcontroller Architecture

Microcontrollers expose hardware functionality through **memory-mapped registers, buses, and interrupt controllers**.

### Microcontroller Fundamentals

- [[General MCU Info]]
    
- [[STM32F446RE Specs]]
    
- [[Registers General MCU]]
    

### Interrupt Architecture

- [[Vector Table NVIC]]
    
- [[NVIC]]
    

### Bus Architecture

- [[AHB Advanced High Performance Bus]]
    
- [[APB Advanced Peripheral Bus]]
    

---

# Clock System

Clock configuration determines **system timing, peripheral speeds, and power consumption**.

### Core Clock References

- [[System Clock Tree]]
    
- [[Clock Frequency for MCU]]
    

### Clock Sources

- [[HSI High Speed Internal Oscillator]]
    
- [[HSE High Speed External Oscillator or Crystal]]
    

### Clock Synthesis

- [[PLL Phase Locked Loops]]
    
- [[PLL HSI HSE relationship]]
    

### Clock Configuration

- [[Prescalar]]
    

### Peripheral Clock Control

- [[Peripheral Clocks]]
    
- [[RCC Reset Clock Control]]
    
- [[RCC_CR Register]]
    
- [[RCC_CFGR Register]]
    
- [[RCC_PLLCFGR Register]]
    
- [[RCC_AHB1ENR Register]]
    
- [[RCC_APB1ENR Register]]
    
- [[RCC_APB2ENR Register]]
    

---

# GPIO

General Purpose IO is controlled through a **set of configuration registers defining pin behavior**.

### GPIO Registers

- [[GPIO Registers]]
    
- [[GPIO MODER]]
    
- [[GPIO OTYPER]]
    
- [[GPIO OSPEEDR]]
    
- [[GPIO PUPDR]]
    
- [[GPIO IDR]]
    
- [[GPIO ODR]]
    
- [[GPIO BSRR]]
    

### HAL GPIO Functions

- [[HAL_GPIO_WritePin]]
    
- [[HAL_GPIO_TogglePin]]
    

---

# Timers and Timekeeping

Timers provide **scheduling, delays, and system timekeeping**.

### System Timer

- [[SYSTick]]
    
- [[HAL_SYSTICK_IRQ_Handler]]
    
- [[HAL_IncTick]]
    

### Timer Architectures

- [[Timer with SYSTick]]
    
- [[Multiple Timers with SYSTick]]
    
- [[Caller Owned Timers]]
    
- [[MultiTimer Observations]]
    

### Time Concepts

- [[Monotonic Counters]]
    
- [[Timer Drift]]
    

---

# Communication and Data Transfer

Peripheral communication and efficient memory transfers.

### UART

- [[UART Universal Asynchronous Receiver or Transmitter]]
    
- [[HAL_UART_Transmit]]
    
- [[HAL_UART_Transmit_DMA]]
    

### DMA

- [[DMA_Pattern]]
    

---

# Embedded Software Architecture

### Logging

- [[Embedded Logging]]
    
- [[Compile Time Logging]]
    
- [[Ring Buffer - Compile Logging Time Relationship]]
    

### Data Structures

- [[Embedded Ring Buffer]]
    
- [[RingBuffer Mask Pattern]]
    

### Firmware Structure

- [[Embedded Workflow]]
    
- [[Project Structure Cohabiting CubeMX]]
    
- [[CubeMX ReStructure File from Generated code]]
    

---

# HAL Layer

The **Hardware Abstraction Layer (HAL)** provides standardized APIs for interacting with hardware peripherals.

- [[HAL Library]]
    
- [[HAL_GPIO_WritePin]]
    
- [[HAL_GPIO_TogglePin]]
    
- [[HAL_UART_Transmit]]
    
- [[HAL_UART_Transmit_DMA]]
    
- [[HAL_IncTick]]
    
- [[HAL_SYSTICK_IRQ_Handler]]

- [[HAL_UART_TxCpltCallback]]

---

# Debugging and Tooling

Tools and workflows for debugging embedded firmware.

- [[Debugging With GDB]]
    
- [[OpenOCD]]
    

---

# Electronics Fundamentals

Basic electrical components and principles used in embedded hardware design.

### Components

- [[Resistor]]
    
- [[Capacitors]]
    
- [[Inductors]]
    
- [[Transistor]]
    

### Electrical Theory

- [[Current]]
    
- [[Ohms Law]]
    
- [[Circuit]]
    

---

# Embedded Concepts

General embedded programming and system concepts.

- [[WatchDOG Timers (WDT)]]
    
- [[uint8 on Embedded]]
    
- [[uint16 and uint8 in Embedded]]
    

---

# Learning and Projects

Personal learning material and projects.

- [[STM32 Learning]]
    
- [[Embedded Online Resources]]
    
- [[Hollix]]