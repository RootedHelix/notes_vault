Related to [[Embedded]] ,  [[packets]]
#embedded #uart #synchronous_protocol

## 🔌 What is UART?

**UART** stands for **Universal Asynchronous Receiver/Transmitter**.

It’s a **hardware communication protocol** used by microcontrollers (like your STM32) to send and receive serial data.

In simple terms:

> UART lets two devices talk to each other over two wires using serial communication.

---

## 📡 What “Serial Communication” Means

Instead of sending 8 bits at once (parallel), UART sends:

```
1 0 1 0 1 1 0 0
```

**One bit at a time**, very fast.

---

## 🧠 Why It’s Called “Asynchronous”

Unlike protocols that use a shared clock (like SPI or I2C), UART:

- ❌ Has **no clock line**
    
- ✅ Relies on both devices agreeing on a **baud rate** (speed)
    

Common baud rates:

- 9600
    
- 115200 (very common in embedded systems)
    

Both sides must match.

---

## ⚙️ Basic UART Wiring

![Image](https://datacapturecontrol.com/articles/static/images/data-communication/wired/uart/intro/uart-intro-title-image.svg)

![[uart_visual.jpg]]

![Image](https://community.st.com/t5/image/serverpage/image-id/96932iEEF6BEEF6FC4CC5F/image-size/large/is-moderation-mode/true?px=999&v=v2)

Minimum connections:

|Wire|Purpose|
|---|---|
|TX|Transmit|
|RX|Receive|
|GND|Shared ground|

**Important rule:**

```
Device A TX → Device B RX
Device A RX → Device B TX
```

They cross.

---

## 📦 What UART Frame Looks Like

When you send a single byte (8 bits), it’s wrapped like this:

```
Start Bit | Data Bits | Optional Parity | Stop Bit(s)
```

Example:

```
0 | 10101010 | 1
```

- Start bit = tells receiver “data is coming”
    
- Data bits = your actual byte
    
- Stop bit = tells receiver “done”
    

---

## 🔬 In Your STM32 World

On your **STM32F446RE**, UART is implemented using peripherals called:

- **USART1**
    
- **USART2**
    
- etc.
    

(USART = Universal Synchronous/Asynchronous Receiver/Transmitter — same thing but supports clocked mode too.)

You’d typically:

1. Enable clock for USART
    
2. Configure GPIO pins to alternate function
    
3. Set baud rate
    
4. Enable TX/RX
    
5. Send data via register or interrupt
    

Example bare-metal send:

```c
USART2->DR = 'A';
```

---

## 💻 Real-World Uses

UART is used for:

- Debugging (printf over serial)
    
- Communicating with:
    
    - GPS modules
        
    - Bluetooth modules
        
    - GSM modules
        
    - Sensors
        
- Talking to a PC via USB-to-Serial adapter
    

When you open a terminal like:

```
screen /dev/ttyUSB0 115200
```

You’re using UART.

---

## 🆚 UART vs Other Protocols

|Protocol|Clock?|Wires|Speed|Complexity|
|---|---|---|---|---|
|UART|No|2|Medium|Simple|
|I2C|Yes|2|Slower|Addressed bus|
|SPI|Yes|3–4|Fast|Master/slave|

UART is the simplest.

---

## 🎯 For You (Embedded Progression)

Since you're working bare-metal:

Next logical exercises:

- Send "Hello World" over UART to PC
    
- Receive a byte and toggle LED
    
- Implement interrupt-driven UART
    
- Implement ring buffer for RX
    

That’s real firmware engineering territory.

---

If you want, I can next:

- Walk through how to configure USART2 on STM32F446RE bare-metal
    
- Or explain how baud rate math works internally