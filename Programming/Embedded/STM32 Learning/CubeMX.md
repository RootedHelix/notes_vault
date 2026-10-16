CubeMX is a software developed by the STM creators and should be used in this manner: 

STM32CubeMX is:

- A clock tree configurator
    
- A peripheral pin mapper
    
- A startup file generator
    
- A HAL initialization code generator


# The Professional Way To Use CubeMX

Here’s how real engineers use it:

### 1️ Use CubeMX to:

- Configure clocks
    
- Configure GPIO
    
- Enable peripherals
    
- Generate base project
    

### 2️ Then:

- Move generated hardware code into `platform/`
    
- Keep Cube-generated code isolated
    
- Never write application logic inside Cube sections
# Long-Term Path

Stage 1: CubeMX + HAL  
Stage 2: HAL wrapped in your own drivers  
Stage 3: LL drivers  
Stage 4: Direct register programming

That’s the natural evolution.