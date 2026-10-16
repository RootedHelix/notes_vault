Related to [[Embedded Logging]], [[Embedded]], [[DMA_zero_copy_Pattern]]
#embedded #embedded_logging #dma_nonblocking_logger

## What happens internally

```
# 6. What happens internally

When you call:

hlx_log("Hello\n");

the system performs:

Application  
   ↓  
hlx_log()  
   ↓  
vsnprintf formatting  
   ↓  
ring buffer write  
   ↓  
hlx_log_flush_dma()  
   ↓  
DMA transfer started if idle  
   ↓  
DMA interrupt on completion  
   ↓  
HAL_UART_TxCpltCallback()  
   ↓  
ring buffer consume  
   ↓  
next DMA chunk sent

So the logger becomes a **background transmission engine**.
```