Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_otyper

> Output Type


Controls:

- Push-pull
- Open-drain
    
```C

GPIOA->OTYPER &= ~(1 << 5);  // Push-pull
```