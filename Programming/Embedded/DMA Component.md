Related to [[Embedded]], [[HAL_UART_Transmit_DMA]], [[Coniguring USART2 TX DMA  in the ioc Using CubeMX]] 
#embedded #hal_library #hal_uart_transmit_dma 


## From the [[STM32 Resources]]

_**DMA**_ stands for Direct Memory Access controller.  
DMA is a bus master and system peripheral providing high-speed data transfers between peripherals and memory, as well as memory-to-memory.  
Data can be quickly moved by DMA _**without any CPU action**_, keeping CPU resources free for other operations.  

This article uses the STM32L476 device as an example. The STM32L476 device embeds 2 DMAs: DMA1 and DMA2.  

Each channel is dedicated to managing memory access requests from one or more peripherals. The two DMA controllers have 14 channels in total. Each channel has an arbiter to handle priority between DMA requests.
