Related to [[Embedded]], [[Timer with SYSTick]]
#embedded #timer_drift

Excellent — this is a _very_ important subtle detail.

You’re asking why we do:

```c
t->last += t->period;
```

instead of:

```c
t->last = now;
```

This is about **timing accuracy and drift**.

---

## The Short Answer

We use:

```c
t->last += t->period;
```

to keep the timer aligned to a fixed schedule.

If we use:

```c
t->last = now;
```

the timer slowly drifts over time.

---

# 🎯 Let’s See The Difference

Assume:

- Period = 1000 ms
    
- The check runs slightly late each time
    
- Code execution + loop takes 3 ms extra
    

---

## ❌ Case 1: `t->last = now;`

Start at 0 ms.

|Event|Time Triggered|
|---|---|
|1|1003 ms|
|2|2006 ms|
|3|3009 ms|
|4|4012 ms|

Each time we reset to “current time”.

The 3 ms delay accumulates.

After many cycles, your timer drifts further and further.

---

## ✅ Case 2: `t->last += t->period;`

Start at 0 ms.

|Event|Scheduled|Actual|
|---|---|---|
|1|1000|1003|
|2|2000|2003|
|3|3000|3003|
|4|4000|4003|

Now the event is always anchored to:

```
0, 1000, 2000, 3000...
```

Even if execution is late, the schedule remains stable.

No accumulated drift.

---

# 🧠 Why This Matters

If you’re:

- Sampling a sensor at 1 kHz
    
- Generating a heartbeat
    
- Running a control loop
    
- Doing PWM-like scheduling
    

Drift destroys timing precision.

In control systems, drift can cause instability.

---

## What’s Actually Happening Mathematically

Using:

```c
t->last = now;
```

You are measuring:

> "1000 ms since the last time I checked"

Using:

```c
t->last += period;
```

You are enforcing:

> "Run exactly every 1000 ms on a fixed timeline"

That’s a huge difference.

---

# ⚡ Even More Important Edge Case

Suppose your loop stalls for 3 seconds.

If you use:

```c
t->last = now;
```

You silently skip missed periods.

If you use:

```c
t->last += period;
```

You can detect that multiple periods passed and potentially catch up.

Professional schedulers depend on this behavior.

---

# 🧠 Industry Insight

RTOS kernels use the same concept.

They schedule tasks relative to:

```
previous_wakeup_time + period
```

Not:

```
current_time
```

Because deterministic periodic behavior matters.

---

# 🎯 Rule of Thumb

Use:

```c
t->last += t->period;
```

For periodic timers.

Use:

```c
t->last = now;
```

For one-shot timers or timeout tracking.

---

# 🧩 When Would You Use `= now`?

Example: timeout waiting for UART response.

You don't care about alignment.  
You just care about elapsed time.

So this is fine:

```c
if ((now - start) > timeout)
```

---

# 🧠 Final Mental Model

`= now`  
→ “Run again this long after whenever I happened to run.”

`+= period`  
→ “Run at exact scheduled intervals.”

---

This is a small line of code…

…but it separates hobby firmware from professional deterministic systems.

---

If you want, next we can explore:

- What happens if the loop is slower than the period
    
- How to detect and handle missed deadlines
    
- How this becomes a real cooperative scheduler
    

Which direction?