Related to [[Embedded]], [[Multiple Timers with SYSTick]]
#embedded #watchdog_timers

> WatchDog Timer (WDT) is basically a hardware supervisory system whose entire job is "If the software stops behaving correctly, reset the system"


---

## Core Idea

A watchdog is a **separate timer running independently of your code**.

Your firmware must periodically:

> **“Feed” / “Kick” / “Refresh”** the watchdog

If you fail to do that in time → **MCU resets automatically**

---

## Mental Model

Think of it like a dead-man’s switch:

• Firmware alive → keeps feeding watchdog → system runs  
• Firmware hangs → watchdog not fed → watchdog expires → RESET

---

## Why this exists (very important)

Embedded systems run in the real world where bad things happen:

✔ Infinite loops  
✔ Deadlocks  
✔ Corrupted memory  
✔ Peripheral lockups  
✔ EMI / electrical noise  
✔ Unexpected states

Without a watchdog → device freezes forever  
With watchdog → device recovers automatically

This is **critical in production systems**.

---

## Visual Timeline

```
Watchdog Timeout = 100 ms

Feed → Feed → Feed → Feed → OK

Feed → Feed → ❌ HANG → (no feed) → TIMEOUT → RESET
```

---

## What “feeding” means

Feeding is just writing a specific value to a register.

Example (conceptual):

```c
WDT->KR = 0xAAAA;   // Refresh watchdog
```

Not magic — just hardware interaction.

---

## Real Firmware Pattern

Classic embedded loop:

```c
while (1)
{
    run_tasks();

    feed_watchdog();
}
```

If `run_tasks()` ever locks up → watchdog never fed → reset

---

## Why watchdogs are HARDWARE (key insight)

Very important concept:

> **The watchdog runs independently of your broken code**

If firmware crashes:

• Normal timers → dead  
• Normal logic → dead  
• Watchdog → STILL RUNNING

That’s why it works.

---

## Real Problems Watchdogs Solve

Watchdogs are not optional in real devices.

They protect against:

✔ Field failures  
✔ Random lockups  
✔ Rare bugs  
✔ Radiation / noise effects  
✔ Unexpected edge cases

Used everywhere:

• Medical devices  
• Automotive ECUs  
• Industrial controllers  
• IoT devices  
• Aerospace systems

---

## Subtle but important truth

A watchdog does **NOT fix bugs**.

It provides:

> **System recovery**

If your firmware has a rare deadlock once every 3 weeks:

Without WDT → Device dead forever  
With WDT → Device resets → keeps working

Huge reliability difference.

---

## Common Beginner Mistake (very common)

Bad pattern:

```c
while (1)
{
    feed_watchdog();
}
```

Congrats — watchdog now useless.

Why?

Because even broken firmware feeds it.

Correct pattern:

✔ Feed only after system proves it’s healthy

Example:

```c
while (1)
{
    run_scheduler();
    service_comms();
    check_sensors();

    feed_watchdog();   // Only if everything ran
}
```

---

## Even more realistic pattern (professional design)

In serious systems:

Multiple subsystems must report alive:

✔ Tasks set "alive flags"  
✔ Only feed WDT if ALL flags set

Prevents partial failures.

---

## STM32 Reality (since you're using one)

STM32 typically has:

✔ IWDG → Independent Watchdog (most robust)  
✔ WWDG → Window Watchdog (stricter timing)

Most firmware uses **IWDG**.

Runs from its own oscillator → works even if clocks break.

---

## Deep Embedded Insight

A watchdog is fundamentally about:

> **Defensive system design**

Because in embedded:

**Failure is guaranteed eventually**

Watchdog = automatic recovery mechanism.

---

If you want next-level understanding, we can explore:

✔ Independent vs Window watchdog  
✔ How STM32 IWDG actually works  
✔ Timeout calculations  
✔ Common production strategies  
✔ Debugging watchdog resets  
✔ How watchdogs interact with RTOS

This topic is _very_ important professionally.