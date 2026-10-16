Related to [[Embedded]] , [[AHB Advanced High Performance Bus]]  [[System Clock Tree]]
#general_mcu_info

Inside your MCU there is no “magic.”

There are just:

- Clock sources
    
- PLL
    
- Prescalers
    
- Buses (AHB/APB)
    
- Peripherals

Think of the MCU like this:

| Component  | Role                     |
| ---------- | ------------------------ |
| PLL        | Creates high-speed clock |
| AHB        | Feeds CPU + memory       |
| APB        | Feeds peripherals        |
| Prescalers | Control speeds           |

---
# Simple Analogy

Imagine a factory:

- HSE = power plant
    
- PLL = voltage transformer
    
- SYSCLK = main power line
    
- AHB/APB = branch circuits
    
- RCC = the breaker panel
    

You don’t plug machines directly into the power plant.

You route through the panel.