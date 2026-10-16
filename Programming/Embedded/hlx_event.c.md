Related to [[Embedded]], [[hlx_event.h]], [[Hollix Documentation]]
#embedded #event_loop #hollix_documentation #subscriber_producer 

# Personal Notes

The `static hlx_event_handler_t s_handlers[HLX_EVENT_ERROR + 1U];` is essentially an array of function pointers ie 

- **module-level event handlers**
- **driver callbacks**
- **state machine handlers**
- **application logic functions**

which are stored/subscribed in `s_handlers` through the `bool hlx_event_subscribe` function.  


The `hlx_event_queue_t` and `hlx_event_handler_t` work in tandem. The event queue stores events in order, and its API is used to add and remove events when needed. The event handler table is an array of function pointers of type `hlx_event_handler_t`, which uses the event type as the index to select the correct handler function to call.


# AI Response

Below is **production-grade, contract-driven documentation** for your implementation file. It is written to clearly define behavior, constraints, and system guarantees—appropriate for embedded firmware.

---

# **Event System Implementation (`hlx_event.c`)**

## Overview

This module implements a **lightweight, non-blocking event system** based on:

- A **fixed-size ring buffer queue**
    
- A **single-subscriber dispatch model per event type**
    
- A **poll-driven dispatcher (main loop execution)**
    

It enables decoupled communication between producers (e.g., ISRs, drivers) and consumers (application logic).

---

## Architecture

### Data Flow

```text
Producer → hlx_event_post() → Queue → hlx_event_dispatch_*() → Subscriber (handler)
```

---

## Internal Components

### Event Queue

```c
struct hlx_event_queue_s {
    hlx_event_t buf[HLX_EVENT_QUEUE_LEN];
    volatile uint32_t head;
    volatile uint32_t tail;
};
```

#### Properties

- Fixed size: `HLX_EVENT_QUEUE_LEN = 32` (must be power of two)
    
- Ring buffer using **mask-based wraparound**:
    
    ```c
    index = (index + 1) & HLX_EVENT_MASK;
    ```
    
- Lock-free single-producer / single-consumer model (implicit assumption)
    

---

### Handler Table

```c
static hlx_event_handler_t s_handlers[HLX_EVENT_ERROR + 1U];
```

- Indexed directly by `hlx_event_type_t`
    
- Supports **one handler per event type**
    
- `NULL` indicates no subscriber
    

---

## Configuration Constraints

- `HLX_EVENT_QUEUE_LEN` **must be a power of two**
    
- Event types must be contiguous and bounded:
    
    ```c
    HLX_EVENT_NONE < type <= HLX_EVENT_ERROR
    ```
    

---

## API Behavior and Contracts

---

### `hlx_event_init`

```c
void hlx_event_init(void);
```

#### Description

Initializes the event system.

#### Behavior

- Resets queue indices (`head = tail = 0`)
    
- Clears all queue entries
    
- Clears all registered handlers

### Concurrency

At first I assumed that I would be using the same pattern that I have used elsewhere in the [[__disable_irq()]], under the # embedded_pattern which uses [[__get_Primask()]]. However under that pattern the interrupts follow - 
- If interrupts were **enabled before**, they get re-enabled
- If interrupts were **already disabled**, you **don’t accidentally enable them**
Under the current pattern which simply utilizes 
`__disable_irq();`
`__enable_irq();`
This Pattern Guarantees that after this function is returned **interrupts are enabled**.
**However upon further inspection this is not needed**. I should only implement the disabling of the  IRQ's pattern mentioned above if a there is another initialization function in [[hlx_app.c]], that enables IRQ's before it returns.



#### Requirements

- Must be called before any other API
    

---

### `hlx_event_post`

```c
bool hlx_event_post(const hlx_event_t* event);
```

#### Description

Enqueues an event from thread/main context.

#### Behavior

- Copies the event into the queue
    
- Advances `head` index
    
- Returns `false` if:
    
    - `event == NULL`
        
    - queue is full
        

#### Queue Full Condition

```c
(next_head == tail)
```

#### Notes

- Non-blocking
    
- Event is copied (caller retains ownership)
    

---

### `hlx_event_post_isr`

```c
bool hlx_event_post_isr(const hlx_event_t* event);
```

#### Description

ISR-safe event posting interface.

#### Current Implementation

- Directly calls `hlx_event_post`
    

#### ⚠️ Important Constraint

This is **only safe if**:

- Single producer model is enforced **or**
    
- Interrupt preemption does not create race conditions
    

Otherwise:

- Requires critical section or atomic protection
    

---

### `hlx_event_get`

```c
bool hlx_event_get(hlx_event_t* event);
```

#### Description

Dequeues the next event.

#### Behavior

- Copies event from queue into user buffer
    
- Advances `tail` index
    
- Returns `false` if:
    
    - queue is empty
        
    - `event == NULL`
        

---

### `hlx_event_is_empty`

```c
bool hlx_event_is_empty(void);
```

#### Description

Checks if the queue is empty.

#### Behavior

```c
return (head == tail);
```

---

### `hlx_event_subscribe`

```c
bool hlx_event_subscribe(hlx_event_type_t type, hlx_event_handler_t handler);
```

#### Description

Registers a handler for a specific event type.

#### Behavior

- Stores handler in `s_handlers[type]`
    
- Returns `false` if:
    
    - invalid event type
        

#### Constraints

- Only **one handler per event type**
    
- New subscription overwrites previous handler
    

---

### `hlx_event_dispatch_one`

```c
void hlx_event_dispatch_one(void);
```

#### Description

Processes a single event from the queue.

#### Behavior

1. Attempts to dequeue event
    
2. Validates event type
    
3. Invokes registered handler (if present)
    

#### Execution Context

- Intended for **main loop**
    
- Not ISR-safe
    

---

### `hlx_event_dispatch_all`

```c
void hlx_event_dispatch_all(void);
```

#### Description

Processes all queued events.

#### Behavior

- Repeatedly calls `hlx_event_dispatch_one`
    
- Runs until queue is empty
    

---

## Execution Model

- **Producers**
    
    - Main context (`hlx_event_post`)
        
    - ISR context (`hlx_event_post_isr`)
        
- **Consumer**
    
    - Main loop via:
        
        ```c
        hlx_event_dispatch_all();
        ```
        
- **Handlers**
    
    - Executed synchronously in dispatch context
        

---

## Concurrency Model

### Assumptions (Critical)

This implementation assumes:

- Single consumer (main loop)
    
- Either:
    
    - Single producer **or**
        
    - No concurrent modification of `head`/`tail`
        

### Potential Risks

- Race conditions between ISR and main context
    
- Non-atomic updates to indices
    

### Mitigation (if needed)

- Disable interrupts around queue operations
    
- Use atomic operations
    
- Separate ISR-safe queue
    

---

## Performance Characteristics

- O(1) enqueue and dequeue
    
- No dynamic allocation
    
- Constant memory footprint
    
- Efficient wrap via bitmask
    

---

## Limitations

- Single handler per event type
    
- No prioritization
    
- No backpressure handling beyond drop-on-full
    
- No thread-safety guarantees across multiple producers
    
- No event filtering beyond type
    

---

## Summary

This module provides:

- A **deterministic, low-overhead event queue**
    
- A **type-based dispatch mechanism**
    
- A **simple pub/sub model with single subscriber per type**
    

It is suitable for:

- Bare-metal systems
    
- Cooperative schedulers
    
- Low-complexity embedded firmware
    

---

## Suggested Future Improvements

- Multi-subscriber support per event type
    
- ISR-safe queue with lock-free guarantees
    
- Event prioritization
    
- Queue overflow instrumentation
    
- Static analysis of handler execution time
    

---

If you want, the next step would be:

- upgrading this into a **multi-producer ISR-safe queue**, or
    
- evolving it toward a **QP-style active object model** (which would be a strong portfolio-level jump).