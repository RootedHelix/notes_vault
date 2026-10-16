Related to [[Embedded]], [[CubeMX]]
#embedded #cubemx

In this context, **IOC** stands for **I/O Configuration**.

That matches what the file is for: configuring the microcontroller’s **inputs/outputs and peripherals** in CubeMX/CubeIDE. Even though the file ends up holding more than just pins, the name comes from that configuration role.

An **`.ioc` file** in embedded development usually means the **STM32CubeMX / STM32CubeIDE project configuration file**.

It is **not your firmware code itself**.  
It is the file that stores the **MCU and peripheral setup choices** you made in CubeMX, such as:

- selected microcontroller or board
    
- clock configuration
    
- enabled peripherals like GPIO, UART, SPI, I2C, timers
    
- pin assignments
    
- middleware/settings
    
- code generation options
    

So the `.ioc` file is basically the **project’s hardware/config blueprint**.

### In practice

When you open the `.ioc` file in **STM32CubeIDE**, it brings up the graphical configuration view.  
When you change settings there and regenerate code, CubeMX uses that `.ioc` file to recreate or update the generated initialization code like:

- `main.c`
    
- `gpio.c`
    
- `usart.c`
    
- `tim.c`
    
- `stm32xxxx_hal_msp.c`
    

### Why it matters

The `.ioc` file lets you:

- return to your configuration later
    
- regenerate setup code safely
    
- avoid manually rewriting low-level init code
    
- keep the MCU pin/peripheral setup documented in one place
    

### Important caution

If you manually edit **generated sections** of code, regenerating from the `.ioc` file can overwrite those parts.  
That is why people usually keep their own logic in:

- `USER CODE BEGIN / END` blocks
    
- separate application files
    

### Simple way to think about it

- **`.ioc` file** = configuration recipe
    
- **generated C files** = code produced from that recipe
    
- **your app files** = your real project logic
    

If you want, I can also show you exactly how the `.ioc` file fits into the folder structure of a CubeIDE project.