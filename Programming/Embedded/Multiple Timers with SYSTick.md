Related to [[Embedded]], [[Timer with SYSTick]], [[HAL_IncTick]]
#multi_timer_systick

Great question — this is exactly how you start designing clean embedded architecture.

Short answer:

 **No, you would not pass `now` as a pointer.**  
You pass it **by value**.

`uint32_t` is 4 bytes. Passing it by value is cheap and clean.

---

## Why Not Pass a Pointer?

Bad idea:

```c
void timer_check(uint32_t *now)
```

Because:

- You're not modifying `now`
    
- It’s just a timestamp
    
- Passing a pointer adds indirection for no benefit
    
- Makes API harder to read
    

Correct idea:

```c
void timer_check(uint32_t now)
```

Clean.  
Simple.  
No confusion.

---

## The Proper Way To Encapsulate This
[]
Instead of raw variables everywhere, create a timer structure.

---

### Step 1: Define a Timer Struct

```c
typedef struct
{
    uint32_t last;
    uint32_t period;
} SoftTimer;
```

---

### Step 2: Create a Timer Function

```c
bool Timer_Expired(SoftTimer *t, uint32_t now)
{
    if ((now - t->last) >= t->period)
    {
        t->last += t->period;   // prevents drift
        return true;
    }
    return false;
}
```

Notice:

- `now` → passed by value
    
- `SoftTimer *t` → passed by pointer because we modify `last`
    

---

## Using It In `while(1)`

> Multi Timer Pattern
#embedded_pattern

```c
SoftTimer ledTimer = {0, 1000};
SoftTimer sensorTimer = {0, 200};
SoftTimer uartTimer = {0, 5000};

while (1)
{
    uint32_t now = HAL_GetTick();

    if (Timer_Expired(&ledTimer, now))
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);

    if (Timer_Expired(&sensorTimer, now))
        readSensor();

    if (Timer_Expired(&uartTimer, now))
        sendHeartbeat();
}
```


[[MultiTimer Observations]]
Now your main loop is clean and readable.

## Big Embedded Insight

What you just built is basically:

- A cooperative scheduler
    
- Producing multiple periodic signals
    
- Superimposed on one output
    

This is how interference patterns arise in real systems too (think PWM overlays, multi-rate control loops, etc.).


---

## Why This Design Is Correct

We pass:

- `now` by value → because it’s read-only
    
- Timer struct by pointer → because we update its internal state
    

That’s good C design.

---

## ⚡ Why Passing `now` Is Good Design

Instead of calling `HAL_GetTick()` inside the timer function, we do:

```c
uint32_t now = HAL_GetTick();
```

Once per loop.

Why?

- Guarantees consistent time reference for all timers
    
- Slightly more efficient
    
- Makes testing easier (you can fake `now`)
    
- Decouples from HAL (good architecture practice)
    

---

##  What You Just Did

You created:

- A reusable timing module
    
- Clean separation of concerns
    
- A testable component
    
- A scalable software scheduler foundation
    

This is professional embedded structure.

---

## If You Want To Level This Up

Next logical step:

1. Add one-shot timers
    
2. Add timer start/stop
    
3. Build a mini cooperative scheduler
    
4. Integrate with your state machine architecture
    

Which direction do you want to go next?