Related to [[Embedded]], [[Embedded Logging]], [[hlx_event.h]]
#embedded #embedded_logging #event_loop #subscriber_producer

# AI Response

In your event system, **subscribers are registered callback handlers that express interest in specific event types**.

---

## Precise Definition

A **subscriber** is:

> A function (`hlx_event_handler_t`) that is registered to be invoked when a particular `hlx_event_type_t` occurs.

---

## In Your Code Terms

You defined:

```c
typedef void (*hlx_event_handler_t)(const hlx_event_t* event);
```

So a subscriber is concretely:

> A function pointer stored in an internal registry, keyed by `hlx_event_type_t`.

---

## Relationship to “Routing Events to Subscribers”

When an event is dispatched:

1. The system reads:
    
    ```c
    event->type
    ```
    
2. It looks up:
    
    - all handlers subscribed to that type
        
3. It invokes them:
    

```c
handler(event);
```

---

## Example

### Subscriber (Handler)

```c
void on_button_pressed(const hlx_event_t* event) {
    // handle button press
}
```

### Subscription

```c
hlx_event_subscribe(HLX_EVENT_BUTTON_PRESSED, on_button_pressed);
```

---

## Dispatch Flow

When this event is posted:

```c
hlx_event_t ev = {
    .type = HLX_EVENT_BUTTON_PRESSED,
    .source = HLX_EVENT_SRC_BUTTON,
    .timestamp_ms = 1234
};
```

Then:

```c
hlx_event_dispatch_one();
```

Results in:

```c
on_button_pressed(&ev);
```

---

## Conceptual Model

This is a classic **publish–subscribe (pub/sub)** pattern:

- **Producer** → posts event (`hlx_event_post`)
    
- **Subscriber** → registers handler (`hlx_event_subscribe`)
    
- **Dispatcher** → connects them (`hlx_event_dispatch_*`)
    

---

## Important Properties (You Should Document)

### 1. Many-to-One or One-to-Many?

You must define:

- Can multiple handlers subscribe to the same type?
    
- Or only one?
    

---

### 2. Execution Context

Subscribers run:

- in **dispatch context** (likely main loop)
    
- not in ISR (unless explicitly designed)
    

---

### 3. Ordering

If multiple subscribers exist:

- Are they called in registration order?
    
- Undefined?
    

---

### 4. Constraints

Subscribers should:

- be non-blocking
    
- avoid heavy work
    
- possibly defer long operations
    

---

## Tight Definition (Documentation-Ready)

You could phrase it like this:

> Subscribers are registered event handlers (`hlx_event_handler_t`) associated with a specific `hlx_event_type_t`. When an event is dispatched, all handlers subscribed to that event type are invoked with a pointer to the event instance.

---

## Bottom Line

- Subscriber = **registered callback function**
    
- Bound to = **event type**
    
- Invoked during = **dispatch**
    
- Mechanism = **function pointer registry**
    

---

This is a correct and standard embedded architecture pattern—you’re implementing a lightweight event bus.
