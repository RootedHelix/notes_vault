Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_bsrr

> **Bit Reset Register**

![[BSSR_image_1.jpg]]
![[BSSR_image_2.jpg]]
![[BSSR_image_3.jpg]]

This is the **professional way** to change GPIO outputs.

Why?

Because it's **atomic** — no read-modify-write issues.

Lower 16 bits → Set pins  
Upper 16 bits → Reset pins

Example:

```C

GPIOA->BSRR = (1 << 5);        // Set PA5
GPIOA->BSRR = (1 << (5 + 16)); // Reset PA5
```

In real embedded systems, BSRR is preferred over ODR.