Related to [[Embedded]], [[HAL Library]]
#embedded #hal_gpio_togglepin


## Function Signature
`HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);`

## What it does:

It:

    Reads the current output state

    Flips it

    Writes the opposite state

So it is a read-modify-write operation.

