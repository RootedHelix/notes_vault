Related to [[Hollix]], [[Embedded]], [[hlx_timers.c]]
#hlx_timers #timer_embedded



The Current Struct and Enum Declarations: 

```C

typedef enum hlx_timer_type_s hlx_timer_type_t;
typedef struct hlx_timer_s hlx_timer_t;

enum hlx_timer_type_s {
    HLX_TIMER_ONESHOT,
    HLX_TIMER_PERIODIC
};

struct hlx_timer_s {
    uint32_t last;
    uint32_t period;
    hlx_timer_type_t type;
};
```

- The last variable is the last time the timer was triggered 
- The period is the numeric interval at which the timer should be triggered
- The type is either `ONEHSHOT` or `PERIODIC` who's distinction only lies in what happens after the timer expires

- `ONESHOT` Timer fires once then becomes inactive


The current API is as follows: 

```C
void hlx_timer_start(hlx_timer_t* timer, uint32_t period, hlx_timer_type_t type);
bool hlx_timer_expired(hlx_timer_t* timer, uint32_t now);
void hlx_handle_timer_expired(const hlx_event_t* event);
```