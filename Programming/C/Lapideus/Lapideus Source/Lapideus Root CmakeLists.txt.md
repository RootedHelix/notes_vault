Related to [[Lapideus]], [[Lapideus Source]]
#root_cmakelists_txt #lapideus_source

### Source File Registration

When adding new source files to the project, list them explicitly in the appropriate `set()` source list inside `CMakeLists.txt`. These source lists are later passed to the relevant build targets through `add_executable()`, `add_library()`, or `target_sources()`.

This project intentionally avoids using `file(GLOB ...)` or `file(GLOB_RECURSE ...)` to collect source files automatically, due to CMake describing them to be avoided in production.
i.g.

```cmake
file(GLOB_RECURSE CORE_SOURCES "src/core/*.c")
```

Now you must register header and source files in the respective directory locations like one of these: 
```CMake
set(CORE_HEADERS
        src/core/lpd_buf.h
        src/core/lpd_config.h
        src/core/lpd_connection.h
        src/core/lpd_core.h
        src/core/lpd_cycle.h
        src/core/lpd_inet.h
        src/core/lpd_log.h
        src/core/lpd_pool_allocator.h
        src/core/lpd_queue.h
        src/core/lpd_rbtree.h
        src/core/lpd_strings.h
        src/core/lpd_times.h
)
```

