Related to [[Embedded]], [[Embedded Ring Buffer]], [[Compile Time Logging]]
#embedded #embedded_ring_buffer #compile_time_logging_embedded 
#embedded_ring_buffer_vs_compile_time_logging

Great question — this is where embedded logging gets really interesting.

### How they relate

What you've built so far is the **filtering layer** — _what_ gets logged and at what verbosity. A ring buffer is the **transport/storage layer** — _where_ the log goes and how it gets out of your system efficiently.

They work together like this:

```

Your code
  → LOG_DEBUG("sensor: %d", val)        // Filtering (compile-time)
    → ring_buffer_write(formatted_msg)   // Storage (ring buffer)
      → UART DMA / USB / debugger read   // Output (async)
```

### Why not just `printf` to UART directly?

In your current setup, `printf` → UART is **blocking**. On a Cortex-M4 at 115200 baud, printing 80 characters takes ~7ms. If your main loop needs to run every 1ms, you just blew your timing budget by 7x. In a real system this can:

- Miss sensor deadlines
- Cause watchdog resets
- Make timing-sensitive bugs disappear when logging is on (heisenbugs 👻)

### What a ring buffer gives you

```

┌─────────────────────────────────────┐
│           Main Loop (fast)          │
│                                     │
│  LOG_DEBUG("val=%d", x)            │
│       │                             │
│       ▼                             │
│  ┌──────────────┐                   │
│  │ Ring Buffer   │  ← just a memcpy │
│  │ [  |  |  |  ]│     (~1 µs)      │
│  └──────┬───────┘                   │
└─────────┼───────────────────────────┘
          │
          ▼  (asynchronous drain)
   ┌──────────────┐
   │  UART + DMA   │  ← happens in background
   │  or USB CDC   │
   │  or SWO/ITM   │
   └──────────────┘
```

- **Write** is non-blocking — you just copy bytes into a circular buffer (~microseconds)
- **Drain** happens asynchronously via DMA, a low-priority task, or an idle hook
- If logs come faster than output can drain, **oldest entries get overwritten** (you lose old logs, not real-time performance)

### How it would fit into your project

Your logging header would change from calling `printf` directly to writing into the buffer:

```

// In hlx_logging.h — swap the backend
#if LOG_LEVEL >= LOG_LEVEL_DEBUG
#define LOG_DEBUG(fmt, ...) hlx_log_write(LOG_LEVEL_DEBUG, fmt, ##__VA_ARGS__)
#else
#define LOG_DEBUG(fmt, ...) ((void)0)#endif
```

A minimal ring buffer module:

```C

// hlx_ringbuf.h
#ifndef HLX_RINGBUF_H
#define HLX_RINGBUF_H
#include <stdint.h>
#include <stdbool.h>
#define HLX_RINGBUF_SIZE 512  // power of 2 for fast masking
typedef struct {    
uint8_t  buf[HLX_RINGBUF_SIZE];    
volatile uint16_t head;  // writer advances head    
volatile uint16_t tail;  // reader advances tail
} hlx_ringbuf_t;
void     hlx_ringbuf_init(hlx_ringbuf_t* rb);
bool     hlx_ringbuf_write(hlx_ringbuf_t* rb, const uint8_t* data, uint16_t len);
uint16_t hlx_ringbuf_read(hlx_ringbuf_t* rb, uint8_t* dest, uint16_t max_len);
bool     hlx_ringbuf_empty(hlx_ringbuf_t* rb);
#endif
```

See [[uint16 and uint8 in Embedded]] for more information regarding why we use those data types instead of uint32.  

Then the log write function formats and pushes into the ring buffer:

```C

// hlx_logging.c
#include <hlx_logging.h>
#include <hlx_ringbuf.h>
#include <stdio.h>
#include <string.h>
static hlx_ringbuf_t log_rb;
void hlx_log_init(void) {   
 hlx_ringbuf_init(&log_rb);
 }
 void hlx_log_write(int level, const char* fmt, ...) {    
 char tmp[128];  
va_list args;
va_start(args, fmt);
int len = vsnprintf(tmp, sizeof(tmp), fmt, args);
va_end(args);
if (len > 0) {
        hlx_ringbuf_write(&log_rb, (uint8_t*)tmp, (uint16_t)len);
            }}// Call this from main loop or idle — drains buffer to UARTvoid hlx_log_flush(void) {
                uint8_t chunk[64];
                uint16_t n = hlx_ringbuf_read(&log_rb, chunk, sizeof(chunk));
                if (n > 0) {
       HAL_UART_Transmit(&huart2, chunk, n, 10);        // or kick off DMA: HAL_UART_Transmit_DMA(&huart2, chunk, n);
           }}
```

And in your main loop:

```C

void app_run(void) {
    uint32_t now = SYSTICK_getTicks();    
    // ... your timer/watchdog logic ...    
    LOG_DEBUG("loop tick: %lu", (unsigned long)now);    
    // Drain a chunk of logs each loop — non-blocking    
    hlx_log_flush();}
```

### The big picture

| Layer          | What you have           | What ring buffer adds          |
| -------------- | ----------------------- | ------------------------------ |
| **Filtering**  | `#if LOG_LEVEL >= ...`  | (same — unchanged)             |
| **Formatting** | `printf` / `vsnprintf`  | (same — unchanged)             |
| **Storage**    | None (straight to UART) | Ring buffer (decouples timing) |
| **Output**     | Blocking `printf`       | Async drain via DMA/polling    |

### When do you actually need this?

- Logging from **ISRs** (you can _never_ `printf` from an interrupt)
- Loops tighter than **~5ms**
- Any system where **timing matters** (motor control, audio, comms)
- Production firmware that needs post-mortem logs

For your current learning/blinky stage, direct `printf` is totally fine. But now you know exactly where the ring buffer slots in when you're ready for it! 🚀