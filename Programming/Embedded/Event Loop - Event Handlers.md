Related to [[Embedded]], [[Event Loop]], [[hlx_event.c]]
#embedded #event_loop 

# AI Response

They interact in **dispatch**.

## The roles

### Event queue

Stores **pending events**:

- what happened
- in what order
- waiting to be processed

### Handler array

Stores **callbacks**:

- which function should run for each event type

## How they connect

When your code dispatches an event:

1. **remove one event from the queue**
2. inspect its `type`
3. use that `type` as an index into the handler array
4. if a handler exists, call it with that event

## In one sentence

The queue provides the **event**, and the handler array provides the **function to handle that event**.

## Example flow

If the queue contains a `UART_RX` event:

- dispatcher dequeues it
- looks up the `UART_RX` handler in the array
- calls that handler with the event data

## Think of it like this

- **queue** = line of things waiting to be handled
- **handlers array** = routing table from event type to function

## Why both are needed

Without the queue:

- events would be handled immediately and ordering would be harder to manage

Without the handler array:

- you’d have the event, but not a clean way to decide what function should process it

## So the interaction is

**post → queue → dequeue → type lookup → handler call**

If you want, I can draw this as a tiny ASCII diagram.


