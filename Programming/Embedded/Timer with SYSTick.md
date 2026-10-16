Related to [[Embedded]], [[SYSTick]],
#embedded #timer_with_systick

```C
while (1)  
  {     /* USER CODE END WHILE */  
	    /* USER CODE BEGIN 3 */    static uint32_t last = 0;  
	    uint32_t now = SYSTICK_getTicks();  
	  
	    if ((now - last) >= 1000)  
	    {      last = now;  
	      HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);  
	    }  
	}  
    /* USER CODE END 3 */  
}

```

> “Has at least `1000` milliseconds passed since `last`?”

Think Of it like
```
	elapsed_time = current_time - start_time
	if elapsed_time >= required_time
	    do something

```

**Powerful Pattern Because:**
#embedded_pattern
- Run multiple timers at once
    
- Avoid blocking
    
- Build cooperative schedulers
    
- Implement state machines
    
- Handle timeouts cleanly
    

All using the same structure.