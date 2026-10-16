Related to [[CMake]], [[Terminal Commands]]

Run `Cmake` configuration
```bash
cmake -S . -B build
```

Build the project
```bash
cmake --build build
```

What is the difference between these?

- Configuration: `CMake` finds `yyjson` and records which source files and libraries belong to each target.
- Building: The compiler compiles your .c files, then the linker combines the resulting object files and libraries.

