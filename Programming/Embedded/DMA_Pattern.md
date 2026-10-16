Related to [[Embedded]], [[HAL_UART_Transmit_DMA]]
#embedded #hal_library #hal_uart_transmit_dma #embedded_pattern

# DMA Logging Flow

With these functions, your logger works like this:

```
log write  
   ↓  
ring buffer  
   ↓  
peek  
   ↓  
DMA starts  
   ↓  
DMA completes  
   ↓  
consume

```

## Optimization next step 
[[DMA_zero_copy_Pattern]]