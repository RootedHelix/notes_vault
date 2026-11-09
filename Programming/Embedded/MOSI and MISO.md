Related to [[Embedded]], [[UART Universal Asynchronous Receiver or Transmitter]], [[Embedded Protocols]]
#embedded #embedded_protocols 

# MOSI and MISO in SPI Communication

**MOSI (Master Out, Slave In)** and **MISO (Master In, Slave Out)** are data lines used in the **Serial Peripheral Interface (SPI)** communication protocol.

SPI is a synchronous serial communication protocol commonly used in embedded systems to exchange data between a microcontroller and peripheral devices such as sensors, displays, ADCs, and flash memory.

## 1. MOSI — Master Out, Slave In

MOSI is the data line used to transmit information **from the SPI controller (master) to the peripheral (slave)**.

For example, when an STM32 microcontroller sends a command to an SPI flash memory chip:

- The STM32 acts as the master.
    
- The flash memory acts as the slave.
    
- Data travels from the STM32 to the flash memory through MOSI.
    

**Direction:**

```
STM32 (Master)                 Flash Memory (Slave)
      MOSI --------------------------->
                 Data
```

## 2. MISO — Master In, Slave Out

MISO is the data line used to transmit information **from the peripheral (slave) back to the controller (master)**.

For example, when an STM32 requests data from an SPI temperature sensor:

- The STM32 initiates communication.
    
- The sensor returns data through MISO.
    
- The STM32 receives the sensor's response.
    

**Direction:**

```
STM32 (Master)                 Sensor (Slave)
      MISO <---------------------------
                 Data
```

## 3. The Four Standard SPI Signals

A typical SPI connection uses four signals:

|Signal|Full Name|Purpose|
|---|---|---|
|MOSI|Master Out, Slave In|Transfers data from master to slave|
|MISO|Master In, Slave Out|Transfers data from slave to master|
|SCLK / SCK|Serial Clock|Synchronizes data transmission|
|CS / NSS|Chip Select|Selects which peripheral communicates|

### SPI Wiring Example

```
       STM32 MCU                    SPI Peripheral
      (Controller)                     (Target)

        MOSI --------------------------> MOSI
        MISO <-------------------------- MISO
        SCLK --------------------------> SCLK
        CS   --------------------------> CS
```

## 4. Full-Duplex Communication

SPI commonly supports **full-duplex communication**, meaning the controller can transmit and receive data simultaneously.

This is possible because MOSI and MISO are separate data lines.

For example, during one SPI clock cycle:

1. The controller outputs one bit on MOSI.
    
2. The peripheral outputs one bit on MISO.
    
3. Both devices sample incoming bits according to the configured SPI clock phase and polarity.
    

This allows SPI to exchange data in both directions during the same transfer.

## 5. Modern Terminology

Modern SPI documentation increasingly uses terminology that avoids master/slave naming.

|Traditional|Modern Alternative|
|---|---|
|MOSI|COPI — Controller Out, Peripheral In|
|MISO|CIPO — Controller In, Peripheral Out|
|Master|Controller|
|Slave|Peripheral|

Both naming conventions refer to the same fundamental signal directions. Some hardware documentation also uses SDO (Serial Data Out) and SDI (Serial Data In), which are named from the perspective of the particular device.

## 6. Key Takeaways

- **MOSI:** Controller sends data to the peripheral.
    
- **MISO:** Peripheral sends data to the controller.
    
- **SCLK:** Controller provides the clock that synchronizes communication.
    
- **CS:** Controller selects the peripheral.
    
- **SPI:** Generally supports simultaneous sending and receiving through separate data lines.
    

Unlike UART, SPI uses a dedicated clock signal, making it a synchronous communication interface.