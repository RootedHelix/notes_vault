Related to [[Embedded]], [[CubeMX]]
#embedded #project_structure_cubemx

The key principle: **don't fight CubeMX** — let it own its files, and put your code somewhere it'll never touch.

### Proposed structure

```

F446RE_Test/
├── Core/                        ← CubeMX-managed (don't put your code here)
│   ├── Inc/
│   │   ├── main.h
│   │   ├── stm32f4xx_hal_conf.h
│   │   └── stm32f4xx_it.h
│   └── Src/
│       ├── main.c
│       ├── stm32f4xx_hal_msp.c
│       ├── stm32f4xx_it.c
│       ├── syscalls.c
│       ├── sysmem.c
│       └── system_stm32f4xx.c
│
├── App/                         ← YOUR code lives here (CubeMX never touches this)
│   ├── Inc/
│   │   ├── app.h
│   │   ├── sensor.h
│   │   ├── led.h
│   │   ├── sys_tick.h
│   │   └── timers.h
│   └── Src/
│       ├── app.c
│       ├── sensor.c
│       ├── led.c
│       ├── sys_tick.c
│       └── timers.c
│
├── Drivers/                     ← CubeMX-managed (HAL, CMSIS)
├── cmake/
├── CMakeLists.txt
├── F446RE_Test.ioc
└── startup_stm32f446xx.s
```

### Steps

**1. Create your `App/` directories:**

```

mkdir -p App/Inc App/Src
```

**2. Move your files out of the CubeMX folders:**

```

mv Inc/sys_tick.h Inc/timers.h App/Inc/mv Src/sys_tick.c Src/timers.c App/Src/
```

**3. Update `CMakeLists.txt`:**

```

// ... existing code ...add_executable(${CMAKE_PROJECT_NAME})# Add STM32CubeMX generated sourcesadd_subdirectory(cmake/stm32cubemx)# Link directories setuptarget_link_directories(${CMAKE_PROJECT_NAME} PRIVATE)# Add your application sourcestarget_sources(${CMAKE_PROJECT_NAME} PRIVATE    App/Src/sys_tick.c    App/Src/timers.c    # App/Src/app.c    # App/Src/sensor.c    # App/Src/led.c)# Add include pathstarget_include_directories(${CMAKE_PROJECT_NAME} PRIVATE    App/Inc)// ... existing code ...
```

**4. Update includes in `main.c`** (inside the USER CODE block):

```

// ... existing code .../* USER CODE BEGIN Includes */#include "sys_tick.h"#include "timers.h"// #include "app.h"  ← eventually just this/* USER CODE END Includes */// ... existing code ...
```

**5. Keep `main.c` thin** — eventually it just calls into your app:

```

/* USER CODE BEGIN 2 */app_init();/* USER CODE END 2 */while (1){    /* USER CODE BEGIN 3 */    app_run();    /* USER CODE END 3 */}
```

### The golden rules

|Folder|Who owns it|Safe to edit?|
|---|---|---|
|`Src/`, `Inc/` (Core)|CubeMX|Only inside `USER CODE` blocks|
|`App/`|You|Always — CubeMX doesn't know it exists|
|`Drivers/`|CubeMX|Never touch|
|`CMakeLists.txt`|You|Yes — CubeMX doesn't overwrite it with CMake projects|

This way you can regenerate from CubeMX any time without worrying, and your application code has a clean, scalable home.