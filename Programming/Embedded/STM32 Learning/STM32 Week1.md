Related to [[Embedded]] [[C]]
#embedded #C #stm32_week1

Goals for week 1 : 
## **Week 1 – OVERVIEW - MCU Fundamentals (Bare Metal Thinking)**

![https://community.st.com/t5/image/serverpage/image-id/87496i742AEB5BDECB455C?v=v2](https://community.st.com/t5/image/serverpage/image-id/87496i742AEB5BDECB455C?v=v2)

![https://community.st.com/t5/image/serverpage/image-id/57656iF2C42EE491DBC366?v=v2](https://community.st.com/t5/image/serverpage/image-id/57656iF2C42EE491DBC366?v=v2)

![https://stm32world.com/images/thumb/c/c6/STM32_Memory_Detail.png/800px-STM32_Memory_Detail.png](https://stm32world.com/images/thumb/c/c6/STM32_Memory_Detail.png/800px-STM32_Memory_Detail.png)


![[ST link and target MCU.jpg]]
**Goals**

- Blink LED (no `delay()` abuse — use timers)
- Button input with debounce
- Understand clock sources (HSI vs HSE)

**Skills**
- GPIO registers
- Clock tree basics
- Flash vs RAM
- Stack vs heap (important later)

**Checkpoint**

> You can explain why a wrong clock config breaks _everything_




## Project Structure
#embedded_collection_structure

embedded_collection/
├── firmware
│   ├── app
│   ├── build
│   ├── config
│   ├── drivers
│   ├── lib
│   └── platform
│       └── stm32f446re
├── LICENSE
├── README.md
└── stm32
    └── F446RE_Test
        ├── cmake
        │   ├── gcc-arm-none-eabi.cmake
        │   ├── starm-clang.cmake
        │   └── stm32cubemx
        │       └── CMakeLists.txt
        ├── CMakeLists.txt
        ├── CMakePresets.json
        ├── Drivers
        │   ├── CMSIS
        │   │   ├── Device
        │   │   │   └── ST
        │   │   │       └── STM32F4xx
        │   │   │           ├── Include
        │   │   │           │   ├── stm32f446xx.h
        │   │   │           │   ├── stm32f4xx.h
        │   │   │           │   └── system_stm32f4xx.h
        │   │   │           ├── LICENSE.txt
        │   │   │           └── Source
        │   │   │               └── Templates
        │   │   ├── Include
        │   │   │   ├── cachel1_armv7.h
        │   │   │   ├── cmsis_armcc.h
        │   │   │   ├── cmsis_armclang.h
        │   │   │   ├── cmsis_armclang_ltm.h
        │   │   │   ├── cmsis_compiler.h
        │   │   │   ├── cmsis_gcc.h
        │   │   │   ├── cmsis_iccarm.h
        │   │   │   ├── cmsis_version.h
        │   │   │   ├── core_armv81mml.h
        │   │   │   ├── core_armv8mbl.h
        │   │   │   ├── core_armv8mml.h
        │   │   │   ├── core_cm0.h
        │   │   │   ├── core_cm0plus.h
        │   │   │   ├── core_cm1.h
        │   │   │   ├── core_cm23.h
        │   │   │   ├── core_cm33.h
        │   │   │   ├── core_cm35p.h
        │   │   │   ├── core_cm3.h
        │   │   │   ├── core_cm4.h
        │   │   │   ├── core_cm55.h
        │   │   │   ├── core_cm7.h
        │   │   │   ├── core_cm85.h
        │   │   │   ├── core_sc000.h
        │   │   │   ├── core_sc300.h
        │   │   │   ├── core_starmc1.h
        │   │   │   ├── mpu_armv7.h
        │   │   │   ├── mpu_armv8.h
        │   │   │   ├── pac_armv81.h
        │   │   │   ├── pmu_armv8.h
        │   │   │   └── tz_context.h
        │   │   └── LICENSE.txt
        │   └── STM32F4xx_HAL_Driver
        │       ├── Inc
        │       │   ├── Legacy
        │       │   │   └── stm32_hal_legacy.h
        │       │   ├── stm32f4xx_hal_cortex.h
        │       │   ├── stm32f4xx_hal_def.h
        │       │   ├── stm32f4xx_hal_dma_ex.h
        │       │   ├── stm32f4xx_hal_dma.h
        │       │   ├── stm32f4xx_hal_exti.h
        │       │   ├── stm32f4xx_hal_flash_ex.h
        │       │   ├── stm32f4xx_hal_flash.h
        │       │   ├── stm32f4xx_hal_flash_ramfunc.h
        │       │   ├── stm32f4xx_hal_gpio_ex.h
        │       │   ├── stm32f4xx_hal_gpio.h
        │       │   ├── stm32f4xx_hal.h
        │       │   ├── stm32f4xx_hal_pwr_ex.h
        │       │   ├── stm32f4xx_hal_pwr.h
        │       │   ├── stm32f4xx_hal_rcc_ex.h
        │       │   ├── stm32f4xx_hal_rcc.h
        │       │   ├── stm32f4xx_hal_uart.h
        │       │   ├── stm32f4xx_ll_bus.h
        │       │   ├── stm32f4xx_ll_cortex.h
        │       │   ├── stm32f4xx_ll_dma.h
        │       │   ├── stm32f4xx_ll_exti.h
        │       │   ├── stm32f4xx_ll_gpio.h
        │       │   ├── stm32f4xx_ll_pwr.h
        │       │   ├── stm32f4xx_ll_rcc.h
        │       │   ├── stm32f4xx_ll_system.h
        │       │   ├── stm32f4xx_ll_usart.h
        │       │   └── stm32f4xx_ll_utils.h
        │       ├── LICENSE.txt
        │       └── Src
        │           ├── stm32f4xx_hal.c
        │           ├── stm32f4xx_hal_cortex.c
        │           ├── stm32f4xx_hal_dma.c
        │           ├── stm32f4xx_hal_dma_ex.c
        │           ├── stm32f4xx_hal_exti.c
        │           ├── stm32f4xx_hal_flash.c
        │           ├── stm32f4xx_hal_flash_ex.c
        │           ├── stm32f4xx_hal_flash_ramfunc.c
        │           ├── stm32f4xx_hal_gpio.c
        │           ├── stm32f4xx_hal_pwr.c
        │           ├── stm32f4xx_hal_pwr_ex.c
        │           ├── stm32f4xx_hal_rcc.c
        │           ├── stm32f4xx_hal_rcc_ex.c
        │           └── stm32f4xx_hal_uart.c
        ├── F446RE_Test.ioc
        ├── Inc
        │   ├── main.h
        │   ├── stm32f4xx_hal_conf.h
        │   └── stm32f4xx_it.h
        ├── Src
        │   ├── main.c
        │   ├── stm32f4xx_hal_msp.c
        │   ├── stm32f4xx_it.c
        │   ├── syscalls.c
        │   ├── sysmem.c
        │   └── system_stm32f4xx.c
        ├── startup_stm32f446xx.s
        └── STM32F446XX_FLASH.ld

## Default Generated main.c

`int main(void)`
`{`
`HAL_Init();`
  `SystemClock_Config();`
  `MX_GPIO_Init();`

  `while (1)`
  `{`
  `}`
`}`

**What does this do**

| Function               | What It Does                          |
| ---------------------- | ------------------------------------- |
| `HAL_Init()`           | Initializes the HAL library + SysTick |
| `SystemClock_Config()` | Sets system clock tree                |
| `MX_GPIO_Init()`       | Initializes pins you configured       |
| `while(1)`             | Your infinite firmware loop           |
# 02.19.25
Things to cover next - Learn:

- SysTick
    
- Hardware timers (TIM2, TIM3, etc.)
    
- Interrupts
    
- NVIC


## 02.20.25

Here is the ideal structure :


## Progress So far


• Halted the Cortex-M4  
• Read memory-mapped RCC register  
• Verified reset clock tree

This is professional bring-up work.


- Blink LD2 with: 

`HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_SET);`
`HAL_Delay(500);`  
`HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, GPIO_PIN_RESET);`
`HAL_Delay(500);`

[[SYSTick]]
- Configured a hardware timer
    
- Enabled an interrupt
    
- Created a system time base
    
- Built a non-blocking timing mechanism

