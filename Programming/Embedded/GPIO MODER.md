Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_moder


> **Mode Register**

![[GPIO_moder.png]]


Each pin uses 2 bits to select its mode:   

| Bits | Mode               |
| ---- | ------------------ |
| 00   | Input              |
| 01   | Output             |
| 10   | Alternate Function |
| 11   | Analog             |
Example: Set PA5 to output
```C

GPIOA->MODER &= ~(3 << (5 * 2));  // Clear bits 
GPIOA->MODER |=  (1 << (5 * 2));  // Set to output (01)
```

