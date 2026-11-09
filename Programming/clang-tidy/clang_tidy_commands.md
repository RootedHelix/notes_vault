# Clang‑tidy Basic Commands

```bash
# Run clang‑tidy on a single file and print the suggested fixes
clang-tidy source.cpp -- -std=c++17

# Run clang‑tidy on a whole compilation database
clang-tidy -p build

# Automatically apply suggested fixes in place
clang-tidy -p build -fix

# Show diagnostics for a specific module
clang-tidy -p build -checks=modernize-* <file>
```

| Flag | Description |
| --- | --- |
| `-p <build-path>` | Path to the `compile_commands.json` build database. |
| `-checks=<list>` | Comma‑separated list of check names to enable or `*` for all. |
| `-fix` | Apply suggested fixes directly to source files. |
| `-format-style=google` | Format the output according to the specified style. |
| `-warnings-as-errors` | Treat warnings as errors. |
| `-quiet` | Suppress informational messages. |

## Example: Running on a single file

```bash
clang-tidy /home/helix/projects/CLearning/src/Leetcode/longest_palindrome.c -p build --checks=*,-clang-analyzer,-llvm-include-order
```

Explanation of flags:

- `-p build`: path to the compilation database (`compile_commands.json`) in the `build` directory.
- `--checks=*,-clang-analyzer,-llvm-include-order`: enable all checks (`*`), but disable the `clang-analyzer` and `llvm-include-order` checks.
