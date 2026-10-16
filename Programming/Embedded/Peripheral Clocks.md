Related to [[Embedded]], [[HAL Library]], [[System Clock Tree]]
#embedded #peripheral_clocks

## (Local clocks)

When you do something like:

`MX_GPIO_Init();`

Inside that function you’ll see:

`__HAL_RCC_GPIOA_CLK_ENABLE();`

That enables the clock **to that specific peripheral block**.

This does NOT configure system frequency.  
It just powers that hardware block.