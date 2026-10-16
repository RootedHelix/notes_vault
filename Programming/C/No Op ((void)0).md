Related to [[embedded]], [[C]]
#embedded #void_0

> ((void)0) is the syntax for no op through macros 

For example 

```
#if LOG_LEVEL >= LOG_LEVEL_WARN  
#define LOG_WARN(fmt, ...) hlx_log_write(LOG_LEVEL_WARN, "[WARN] " fmt "\r\n", ##__VA_ARGS__)  
#else  
#define LOG_WARN(fmt, ...) ((void)0)  
#endif
```