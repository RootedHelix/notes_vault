Related to [[Embedded]]
#baud_rate #embedded #embedded_logging 

## AI Reponse

> **Baud rate** is the number of **signal changes (symbols) per second** on a communication line.

### Key distinction (this is where most confusion happens)

- **Baud rate** = symbols per second
    
- **Bit rate** = bits per second
    

They are only equal when **1 symbol = 1 bit**.

---

### In UART / embedded systems (your case)

For standard UART communication (like your STM32 using `USART2`):

- Each symbol represents **1 bit**
    
- Therefore:
    

`baud rate = bit rate`

So when you see:

```
115200
```

That means:

- **115,200 bits per second**
    
- The line toggles **115,200 times per second**
    

---

### Practical interpretation (UART frame level)

UART doesn’t just send raw bits—it sends **frames**:

Typical configuration:

- 1 start bit
    
- 8 data bits
    
- (optional parity)
    
- 1 stop bit
    

So for **8N1 (very common)**:

- Total = **10 bits per byte**
    

At **115200 baud**:

`115200 / 10 = 11,520 bytes / sec (max theoretical)`

---

### Why it matters for your setup

When you open `/dev/ttyACM0` with something like `picocom`:

```
picocom -b 115200 /dev/ttyACM0
```

- The **baud rate must match** what your STM32 is configured for
    
- If it doesn’t:
    
    - You’ll see **garbled output**
        
    - Or nothing at all
        

---

### Quick intuition

Think of baud rate as:

> “How fast the wire is changing state”

And bit rate as:

> “How much actual information is being delivered”

---

If you want, I can walk through how STM32 configures this internally (BRR register, oversampling, clock derivation), which is directly relevant to your UART logger.