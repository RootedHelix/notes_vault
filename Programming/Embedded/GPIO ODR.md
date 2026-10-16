Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_odr 

> **Output Data Register**


Write output state:

```C
GPIOA->ODR |= (1 << 5);  // Set HIGH 
GPIOA->ODR &= ~(1 << 5); // Set LOW

```