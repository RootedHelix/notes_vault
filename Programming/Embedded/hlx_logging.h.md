Related to [[Embedded]], [[hlx_logging.c]], [[Embedded Logging]]
#embedded #hlx_logging #embedded_logging 

## Personal Notes 

This header includes compile time definitions for the various levels of logging. Each level of logging is gated by a conditional that compares the `LOG_LEVEL` with the `enum` value of each level. Based upon what `LOG_LEVEL` that is assigned in the `cmakelists.txt` that will determine whether or not each level of logging is even compiled. 

There is also a clamp that states 
```
#ifndef LOG_LEVEL  
#define LOG_LEVEL LOG_LEVEL_DEBUG  
#endif
```
which essentially states that if the LOG_LEVEL is not already defined then it would be macro'd to LOG_LEVEL_DEBUG which equates to 4 in the current enumeration

Using [[No Op ((void)0)]], `((void)0)` we are able to assign that to all the macros that don't satisfy the conditional such as `#if LOG_LEVEL >= LOG_LEVEL_ERROR`.

Using `extern UART_HandleTypeDef huart2;` we are able to 

This architecture allows for the code that is necessary for observation/logging to be toggled in layers depending upon whether the system is going to be in a production environment or in testing.