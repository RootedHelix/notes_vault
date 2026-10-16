In CMake, a _static library_ is a library that is compiled into a single `.a` (Unix) or `.lib` (Windows) file and then **copied into** each executable or shared library that links against it at link time.

### How you declare one

```

add_library(my_lib STATIC    src/foo.c    src/bar.c)
```

Key points:

- `STATIC` here means: “build a static library”.
- The output is typically:
    - `libmy_lib.a` on Linux/macOS
    - `my_lib.lib` on Windows

To use it:

```

add_executable(my_app main.c)target_link_libraries(my_app PRIVATE my_lib)
```

### Static vs shared (dynamic)

- **Static library (`STATIC`)**
    
    - Code is copied into the final binary at link time.
    - No separate `.so`/`.dll` needed at runtime.
    - Increases binary size; each consumer gets its own copy.
    - Changes to the library require relinking the executable.
- **Shared library (`SHARED`)**
    
    - Produces `.so` (Linux), `.dylib` (macOS), or `.dll` + import lib (Windows).
    - Loaded at runtime.
    - Multiple executables can share the same library in memory.
    - Easier to update without rebuilding everything (as long as ABI is stable).

### Related CMake options

- If you omit the type:

```

  add_library(my_lib src/foo.c src/bar.c)
```

then the type is controlled by `BUILD_SHARED_LIBS`:

- `ON` → shared library
    
- `OFF` → static library (default is often OFF, but project-dependent).
    
- You can mix types in the same project, e.g.:
    

```

  add_library(core_static STATIC core.c)  add_library(core_shared SHARED core.c)
```

If you show your `CMakeLists.txt`, I can point out exactly where a static library is defined and how to link to it.