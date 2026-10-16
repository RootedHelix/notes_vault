Related to [[Embedded]], [[GPIO Registers]]
#embedded #gpio_pupdr

> ** Pull Up / Pull Down** 

Each pin:

|Bits|Meaning|
|---|---|
|00|No pull|
|01|Pull-up|
|10|Pull-down|

Example:

```C

GPIOA->PUPDR |= (1 << (5 * 2)); // Pull-up

```