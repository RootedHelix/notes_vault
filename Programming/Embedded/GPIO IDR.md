Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_idr

> **Input Data Register**

Read pin state:

```c

uint32_t state = (GPIOA->IDR & (1 << 5));

```
