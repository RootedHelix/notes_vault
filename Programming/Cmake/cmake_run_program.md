Related to [[CMake]], [[Terminal Commands]]

clanRun `Cmake` configuration
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

## Run Cmake Program
Below is a step‑by‑step guide to build and run a CMake‑based C program from the terminal, with each flag explained.

***Configuration***
```bash
cmake \
  -S <source_dir>          # Path to the root of your source tree (default: current directory).
  -B <build_dir>           # Path where CMake should store its build files (create if missing).
  -G "Unix Makefiles"      # Generator to use; Ninja is best for Embedded Development; common choice for make, but you can pick Ninja, Xcode, etc.
  -DCMAKE_BUILD_TYPE=Release # Build type: Debug, Release, RelWithDebInfo, MinSizeRel.
  -DCMAKE_INSTALL_PREFIX=/usr/local # Where `make install` will copy files.
  -DSOMEPROJECT=ON          # Pass any project‑specific options you defined.
```

**Explanation of flags**
- `-S`: CMake needs the source directory to find `CMakeLists.txt`.
- `-B`: Keeps build files separate from source; allows out‑of‑source builds and easy clean‑ups.
- `-G`: The generator chooses the build system; Ninja is fast, Makefiles use `make`.
- `DCMAKE_BUILD_TYPE`: Optimises compiler and linker settings for debugging or performance.
- `DCMAKE_INSTALL_PREFIX`: Destination for `cmake --install`.
- `DSOMEPROJECT`: Generic example of passing a BOOL or string option.

***Build***
```bash
cmake --build <build_dir> \ # Target directory created by `-B`
       --target all      \ # Default; can be a specific target like `my_executable`.
       -- -j$(nproc)   # Parallel jobs; `-j` follows the build tool.
```

***Install (optional)***
```bash
cmake --install <build_dir> \ # Copies files to the install prefix.
      --prefix=/usr/local  # Overrides `DCMAKE_INSTALL_PREFIX` if needed.
```

***Run the program***
```bash
# Assuming the executable is in the build output folder
./build/my_executable      # Replace with the actual binary name.
```

---
