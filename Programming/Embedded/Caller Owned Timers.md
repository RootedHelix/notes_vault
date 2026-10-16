Related to [[Embedded]] [[Timer with SYSTick]]
#embedded #timer_pattern #embedded_pattern 

Excellent question — this is **exactly the kind of design decision real embedded engineers wrestle with 👍**

You’ve identified a very important issue:

👉 _Where does timer state actually live?_

Because your current code has a **critical bug**:

---

## 🚨 Problem in your current design

```c
void hlx_timers_init(void)
{
    hlx_timer_t led_timer = {0, 1000};
}
```

Those timers are **local stack variables**.

Meaning:

✔ Created  
✔ Function exits  
💥 Destroyed immediately

All state is lost.

Timers = useless.

Classic C lifetime trap 😄

---

## ✅ Professional Embedded Rule #1

> **Persistent state NEVER lives on the stack**

Timers are **stateful objects** → must persist.

---

## 🚫 Heap allocation? Almost never in embedded

Technically possible:

```c
hlx_timer_t *timer = malloc(sizeof(hlx_timer_t));
```

But in embedded systems:

❌ Heap fragmentation risk  
❌ Allocation failure risk  
❌ Non-deterministic timing  
❌ Many systems forbid malloc entirely

Most firmware:

👉 **Avoids heap for core runtime objects**

Especially timers.

---

## ✅ Professional Embedded Approaches (ordered by sanity)

---

# ✅ **Approach 1 — Static Storage (MOST COMMON)** ✅

Timers live in `app.c`.

Persist forever.

---

### **app.c**

```c
static hlx_timer_t led_timer;
static hlx_timer_t button_timer;
static hlx_timer_t sensor_timer;
static hlx_watchdog_t sensor_wdt;
```

---

### **hlx_timers_init()**

```c
void hlx_timers_init(void)
{
    led_timer    = (hlx_timer_t){0, 1000};
    button_timer = (hlx_timer_t){0, 2000};
    sensor_timer = (hlx_timer_t){0, 3000};

    sensor_wdt   = (hlx_watchdog_t){0, 10000, false};
}
```

✔ No heap  
✔ No copies  
✔ No lifetime issues  
✔ Deterministic  
✔ Extremely standard

This is how most firmware is written.

---

# ✅ **Approach 2 — Return Struct by Value (VALID, less common)**

Modern compilers optimize this well.

Example:

---

```c
hlx_timer_bundle_t hlx_timers_init(void)
{
    return (hlx_timer_bundle_t) {
        .led_timer    = {0, 1000},
        .button_timer = {0, 2000},
        .sensor_timer = {0, 3000}
    };
}
```

Then:

```c
timers = hlx_timers_init();
```

✔ Clean  
✔ Safe  
✔ Used sometimes

But…

Embedded engineers often avoid this when objects grow large.

---

# ✅ **Approach 3 — Caller Provides Storage (VERY PROFESSIONAL)** ⭐

Extremely common in embedded libraries.

---

### Function signature

```c
void hlx_timers_init(hlx_timer_t *led,
                     hlx_timer_t *button,
                     hlx_timer_t *sensor);
```

---

### Caller owns memory

```c
static hlx_timer_t led_timer;

hlx_timers_init(&led_timer, ...);
```

✔ No heap  
✔ Explicit ownership  
✔ Scales well  
✔ Library-friendly

Very robust design.

---

## ✅ What professionals almost always choose

For **application-level firmware objects**:

👉 **Static storage**

For **reusable modules/libraries**:

👉 **Caller-provided storage**

Heap = rare / restricted.

---

## ✅ Why static storage is king in embedded

Timers are:

✔ Stateful  
✔ Always needed  
✔ Long-lived  
✔ Deterministic timing critical

Static = perfect fit.

---

## ✅ Clean architecture for YOUR project (ideal)

---

### **app.c**

```c
static hlx_timer_t led_timer;
static hlx_timer_t button_timer;
static hlx_timer_t sensor_timer;
```

---

### **app_init()**

```c
void App_Init(void)
{
    hlx_timers_init();

    led_timer.period = 1000;
}
```

---

### **app_run()**

```c
void App_Run(void)
{
    uint32_t now = HAL_GetTick();

    if (hlx_timer_expired(&led_timer, now))
    {
        HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
    }
}
```

✔ Very professional  
✔ Very scalable  
✔ Very deterministic  
✔ Matches your FSM mindset

---

## 🎯 Key Embedded Insight You Just Hit

You’ve crossed into:

👉 **State Lifetime & Ownership Design**

This is core embedded engineering thinking.

Not beginner stuff anymore 👍

---

If you want next extremely useful step:

👉 Designing a clean timer API (this is a great exercise)  
👉 Event-driven scheduling model  
👉 Cooperative scheduler design  
👉 FSM-driven timing system

Timers + FSM = where embedded gets fun 😄