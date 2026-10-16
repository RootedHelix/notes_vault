Related to [[Embedded]], [[Embedded Logging]],
#embedded #embedded_logging #logging_policy

The System should separate different layers of responsibility when it comes to logging. 

Given these layers: 
``` 
#define LOG_LEVEL_NONE  0
#define LOG_LEVEL_ERROR 1
#define LOG_LEVEL_WARN  2
#define LOG_LEVEL_INFO  3
#define LOG_LEVEL_DEBUG 4
```

Where `hlx_app.c` _should_ log

Only log **system-level semantics**, not internal mechanics:

Good:
```C
LOG_INFO("System initialized");  
LOG_INFO("Entering main event loop");  
LOG_WARN("High event queue latency detected");  
LOG_ERROR("System entering safe mode");
```

Bad (this belongs in modules):

```C
LOG_DEBUG("Timer tick incremented");     // hlx_sys_tick.c  
LOG_DEBUG("Event pushed to queue");      // hlx_event.c
```