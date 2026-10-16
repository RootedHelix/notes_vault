Related to [[Embedded]], [[HAL Library]]
#embedded #hal_gpio_writepin

## Function Signature

`void HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState  PinState)`

PinState can bring one of the two: 

`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);`
`HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);`

## What it does:



It **forces** the pin to a specific state.

#enum_gpio_pinstate
`enum GPIO_PinState` 
{ 
- `GPIO_PIN_RESET` → output LOW
- `GPIO_PIN_SET` → output HIGH 

}

Under the hood, it writes to the **BSRR register**, which is atomic.

That means:

- It does NOT read the current state.
    
- It directly sets or clears the bit.
    
This is safe and deterministic.


## When To Use Each

#hal_gpio_writepin_vs_hal_gpio_togglepin
### Use `HAL_GPIO_WritePin()` when:

- You want deterministic control
    
- You’re implementing logic like:
    
    - Turn LED on when button pressed
        
    - Turn relay off on fault
        
- You care about exact state
    

### Use `HAL_GPIO_TogglePin()` when:

- Simple blinking
    
- Debug heartbeat LED
    
- No other code modifies that pin
    

---