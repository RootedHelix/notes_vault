Related to [[Embedded]], [[Embedded Logging]], [[DMA Concurrency issues]], [[HAL_UART_Transmit_DMA]]

#embedded #embedded_logging #embedded_pattern 

## 03.13.25
[[2026-03-13]]

This is the current structure: 

```C
void hlx_log_flush_dma(void) {  
    __disable_irq();  
  
    if (log_tx_busy) {  
        __enable_irq();  
        return;  
    }  
  
    uint8_t log_dma_buf[HLX_LOG_CHUNK_SIZE];  
    //    uint32_t n = hlx_ringbuf_peek(&log_rb, log_dma_buf, sizeof(log_dma_buf));  
    uint8_t* data;  
    uint32_t n = hlx_ringbuf_linear_read(&log_rb, &data);  
  
    if (n == 0U) {  
        __enable_irq();  
        return;  
    }  
    if (n > HLX_LOG_CHUNK_SIZE) {  
        n = HLX_LOG_CHUNK_SIZE;  
    }  
    log_tx_busy = true;  
    log_dma_len = n;  
  
    __enable_irq();  
  
    if (HAL_UART_Transmit_DMA(&huart2, data, n) != HAL_OK) {  
        __disable_irq();  
        log_tx_busy = false;  
        log_dma_len = 0U;  
        __enable_irq();  
    }
}
```

**In regards to concurrency**

the idea is that I need to snapshot the data at that point in time because the ringbuffer memory might change in the time from me doing a linear read to me doing the HAL_UART_Transmit

The window from hlx_ringbuf_linear_read() and HAL_UART_Transmit_DMA() is what is risky.
In addition the more important window is from: 
- the moment you hand the pointer to DMA
- until the DMA transfer actually finishes
