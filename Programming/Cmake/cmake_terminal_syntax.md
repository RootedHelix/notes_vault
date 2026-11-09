:Related to [[CMake]], [[Terminal Commands]]

## Build

clanRun `Cmake` configuration
```bash
cmake -S . -B build
```

`-S` is the source directory. `.` means the current directory, so CMake reads `CMakeLists.txt` from the directory you ran the command in. Run this from the project root.

`-B` is the build directory. `build` is the folder CMake creates for generated files: the cache, Makefiles or Ninja files, object files, and the executable. Source stays in the project tree; build output goes in `build/`.

###### Build the project

```bash
cmake --build build
```

**Target Certain Directory**

```bash
cmake --build build --target embedded
```

Use the `--target` flag to specify a specific target for compilation.


#### Explanation

What is the difference between these?

- Configuration: `CMake` finds `yyjson` and records which source files and libraries belong to each target.
- Building: The compiler compiles your .c files, then the linker combines the resulting object files and libraries.

## Run Program

1.
```bash
Navigate to Root Directory:
cmake -G "Ninja"
```

2.
```bash
Navigate to the build Directory and find the executable:
./executable_name
```

See [[cmake_run_program]] for more information.
