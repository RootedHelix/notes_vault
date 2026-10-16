Related to [[Embedded]], [[Hollix]], [[Event driven vs Hyperthreading]]
#embedded #hollix #event_driven_vs_hyprthreading

# Personal Notes

Enumeration for `hlx_event_type_t` which is the enumeration used to define all the types of events the system holds. The second enumeration is `hlx_event_source_t` which is from what subsystem the event stems from. 

Then we have the uinion which holds various structs that hold different structs with data pertaining to the different kinds of events in the system ie `timer, gpio, sensor, error`

The higher level struct of this header is the `hlx_event_s/t` struct which is defined as follows: 
```C
struct hlx_event_s {  
    hlx_event_type_t    type;  
    hlx_event_source_t  source;  
    uint32_t            timestamp_ms;  
    hlx_event_data_t    data;  
};
```

This struct holds objects for the two enumerations and he struct previously mentioned with an additional variable for the timestamp of the event measured in mili-seconds.

### Function Prototypes

`void hlx_event_init(void);`  - Initialize the event struct 

`bool hlx_event_post(const hlx_event_t* event);` - Pushes the event to the event queue
`bool hlx_event_post_isr(const hlx_event_t* event);` - 

`bool hlx_event_get(hlx_event_t* event);` - 
`bool hlx_event_is_empty(void);` - 

`bool hlx_event_subscribe(hlx_event_type_t type, hlx_event_handler_t handler);` - 
`void hlx_event_dispatch_one(void);` - 
`void hlx_event_dispatch_all(void);` - 
``

The `enum hlx_event_type_t`  should be tracking what events are going to take place and order can matter it just depends how you decide to use the enum. 

# AI Documentation

Below is a tightened, production-grade revision with corrections and improved specificity.

---

## Structural Documentation (Rewritten)

### Overview

This module defines the core event abstraction used by the system, including:

- Event classification (`hlx_event_type_t`)
    
- Event origin (`hlx_event_source_t`)
    
- Event payload (`hlx_event_data_t`)
    
- Event container (`hlx_event_t`)
    
- Event queueing and dispatch interface
    

---

### Event Types

`hlx_event_type_t` enumerates all event categories recognized by the system. Each value represents a distinct semantic event that can be produced and consumed.

Examples:

- `HLX_EVENT_TIMER_EXPIRED`
    
- `HLX_EVENT_BUTTON_PRESSED`
    
- `HLX_EVENT_UART_RX`
    
- `HLX_EVENT_ERROR`
    

This enumeration is used for:

- Routing events to subscribers
    
- Determining how payload data should be interpreted
    

---

### Event Sources

`hlx_event_source_t` identifies the subsystem that generated the event.

Examples:

- `HLX_EVENT_SRC_TIMER`
    
- `HLX_EVENT_SRC_SENSOR`
    
- `HLX_EVENT_SRC_ISR`
    

This enables:

- Debug tracing
    
- Filtering or prioritization by subsystem
    
- Separation of concerns in handlers
    

---

### Event Payload (`hlx_event_data_t`)

A tagged union containing payloads for different event categories.

Each member corresponds to a subset of event types:

- `timer` → used with `HLX_EVENT_TIMER_EXPIRED`
    
- `gpio` → used with button or GPIO events
    
- `sensor` → used with sensor-related events
    
- `error` → used with `HLX_EVENT_ERROR`
    

⚠️ **Contract:**  
The `type` field of `hlx_event_t` determines which union member is valid. Accessing an unrelated member is undefined behavior.

---

### Event Structure

```c
struct hlx_event_s {
    hlx_event_type_t    type;
    hlx_event_source_t  source;
    uint32_t            timestamp_ms;
    hlx_event_data_t    data;
};
```

Represents a single event instance in the system.

Fields:

- **type**: Logical classification of the event
    
- **source**: Originating subsystem
    
- **timestamp_ms**: Time of event creation in milliseconds (system time base dependent)
    
- **data**: Payload associated with the event
    

---

## API Documentation (Rewritten with Contracts)

### Initialization

```c
void hlx_event_init(void);
```

Initializes the event subsystem.

- Must be called before any other API
    
- Initializes internal queue and subscriber registry
    

---

### Event Posting

```c
bool hlx_event_post(const hlx_event_t* event);
```

Posts an event to the event queue from thread/main context.

- Returns `false` if the queue is full
    
- Copies event data into the queue (caller retains ownership)
    

---

```c
bool hlx_event_post_isr(const hlx_event_t* event);
```

Posts an event from interrupt context.

- ISR-safe (non-blocking, no dynamic allocation)
    
- Typically uses lock-free or interrupt-masked enqueue
    
- Returns `false` if queue is full
    

⚠️ Must be bounded and deterministic—no heavy processing

---

### Event Retrieval

```c
bool hlx_event_get(hlx_event_t* event);
```

Retrieves the next event from the queue.

- Returns `false` if queue is empty
    
- Copies event into provided buffer
    

---

```c
bool hlx_event_is_empty(void);
```

Checks whether the event queue is empty.

---

### Subscription

```c
bool hlx_event_subscribe(hlx_event_type_t type, hlx_event_handler_t handler);
```

Registers a handler for a specific event type.

- Multiple handlers per type may or may not be supported (implementation-defined — you should document this explicitly)
    
- Returns `false` on failure (e.g., no space in subscriber table)
    

---

### Dispatch

```c
void hlx_event_dispatch_one(void);
```

- Retrieves one event from the queue
    
- Invokes all registered handlers for that event type
    

---

```c
void hlx_event_dispatch_all(void);
```

- Processes the entire event queue
    
- Typically called in the main loop
    

---

## Critical Improvements You Should Add

### 1. Execution Model (Missing — Important)

Explicitly state:

- Is dispatch done in main loop only? (likely yes)
    
- Are handlers allowed to block? (should be no or tightly controlled)
    
- Are handlers re-entrant?
    

Example:

> Event handlers execute in the main loop context and must be non-blocking. Long-running operations should be deferred.

---

### 2. ISR vs Main Context Boundary

You correctly separated:

- `hlx_event_post`
    
- `hlx_event_post_isr`
    

But you should explicitly document:

- Whether `hlx_event_get` / dispatch are ISR-safe (they should NOT be)
    
- Whether interrupts are masked internally
    

---

### 3. Queue Semantics (Currently Missing)

Define:

- Queue size (fixed? configurable?)
    
- Overflow behavior (drop newest? drop oldest?)
    
- Ordering guarantees (FIFO assumed)
    

---

### 4. Memory Model

State clearly:

- Events are **copied**, not referenced
    
- No dynamic allocation
    
- Safe for static/stack-allocated events
    

---

### 5. Type Safety Observation (Advanced)

Right now:

- `type` and `data` are loosely coupled
    

A more advanced note (not required yet, but worth understanding):

You are implementing a **tagged union without enforcement**.

In higher-assurance systems, you'd:

- Add validation
    
- Or wrap creation in helper constructors
    

---

## Bottom Line Assessment

This is **solid junior-to-mid embedded architecture work**, but:

- Your implementation direction is good
    
- Your documentation needs to become **contract-driven**, not descriptive
    
- You need to explicitly define **behavior under constraints** (ISR, queue limits, execution model)
    

If you refine documentation like this consistently, it will read as **professional firmware-level work**, not just learning notes.

---

If you want next step: I can help you evolve this into an **NGINX-style event core** or a **QP-lite style active object system**, depending on how far you want to push it.
