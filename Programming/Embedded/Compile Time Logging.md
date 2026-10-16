Related to [[Embedded]], [[Embedded Logging]], [[Embedded Ring Buffer]]
#embedded #compile_time_logging_embedded #embedded_ring_buffer #embedded_pattern 

Excellent — this is where architecture discipline shows up.

There are **three professional places** a default log level can live, and they each serve a different purpose.

Let’s go from most correct → least correct.

---

## 1️ Best Practice — Define It in the Build System

This is the cleanest and most scalable solution.

In **CMake**:

```cmake
target_compile_definitions(app PRIVATE LOG_LEVEL=LOG_LEVEL_DEBUG)
```

Or:

```cmake
target_compile_definitions(app PRIVATE LOG_LEVEL=LOG_LEVEL_ERROR)
```

Now:

- The source code never changes
    
- Dev builds and prod builds differ cleanly
    
- CI/CD can control verbosity
    
- No accidental commits with DEBUG enabled
    

This is how professional firmware teams do it.

---

## 2️ Safe Default in `hlx_log.h`

You still want a fallback in case the build system doesn’t define it.

In `hlx_log.h`:

```c
#ifndef LOG_LEVEL
#define LOG_LEVEL LOG_LEVEL_DEBUG
#endif
```

Why this pattern?

Because:

- If build system defines it → that wins
    
- If not → you get a sane default
    
- No redefinition warnings
    
- Portable across toolchains
    

This is very common.

---

## 3️ Conditional Based on Build Type

Another clean pattern:

```c
#ifdef DEBUG_BUILD
#define LOG_LEVEL LOG_LEVEL_DEBUG
#else
#define LOG_LEVEL LOG_LEVEL_ERROR
#endif
```

Then in CMake:

```cmake
target_compile_definitions(app PRIVATE DEBUG_BUILD)
```

This ties log level to build configuration.

---

## What You Should NOT Do

Don’t put this in a random `.c` file:

```c
#define LOG_LEVEL LOG_LEVEL_DEBUG
```

Why?

Because:

- Other files won’t see it
    
- You get inconsistent behavior
    
- It breaks modularity
    
- Hard to override later
    

Log level must be globally visible at preprocessing time.

---

## Clean Architecture Summary

Your setup should look like:

```
hlx_log.h
```

```c
#ifndef LOG_LEVEL
#define LOG_LEVEL LOG_LEVEL_DEBUG
#endif
```

And your build system controls it when needed.

That gives you:

|Scenario|Behavior|
|---|---|
|Dev build|Verbose|
|Production|Errors only|
|Field debug firmware|Info level|
|Manufacturing firmware|Possibly none|

All without changing code.

---

## Since You're Using CMake (Important)

You are building like a real embedded dev already.

Your `CMakeLists.txt` probably has:

```cmake
add_executable(F446RE_Test ...)
```

Right after that, you’d add:

```cmake
target_compile_definitions(F446RE_Test PRIVATE LOG_LEVEL=LOG_LEVEL_INFO)
```

Or even better:

```cmake
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(F446RE_Test PRIVATE LOG_LEVEL=LOG_LEVEL_DEBUG)
else()
    target_compile_definitions(F446RE_Test PRIVATE LOG_LEVEL=LOG_LEVEL_ERROR)
endif()
```

Now it automatically switches based on:

```bash
cmake -DCMAKE_BUILD_TYPE=Debug ..
```

That’s professional build control.

---

## Embedded Mindset Shift

You don’t “set debug mode in code”.

You set it in:

> The build configuration.

That’s a big difference from hobby projects.

---

## Final Answer

Define the default in `hlx_log.h` with:

```c
#ifndef LOG_LEVEL
#define LOG_LEVEL LOG_LEVEL_DEBUG
#endif
```

But override it from your build system for real builds.

That gives you:

✔ Clean architecture  
✔ Central control  
✔ Zero runtime cost  
✔ Scalable firmware variants

---

If you want, next I can show you:

- How to structure multiple firmware variants cleanly in CMake
    
- Or how log levels interact with optimization flags (-Os vs -O2)
    
- Or how to completely strip logging symbols from release builds
    

Which direction?