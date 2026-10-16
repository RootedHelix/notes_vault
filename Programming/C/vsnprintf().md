Related to [[C]]

### AI Response

`vsnprintf()` is the `va_list` version of `snprintf()`.

It formats text into a fixed-size character buffer, while preventing buffer overflow.

The function is declared in:

```c
#include <stdio.h>
```

Its signature is:

```c
int vsnprintf(char *str, size_t size, const char *format, va_list ap);
```

## The basic idea

This:

```c
snprintf(buffer, sizeof(buffer), "value = %d", 42);
```

uses normal arguments.

This:

```c
vsnprintf(buffer, sizeof(buffer), "value = %d", args);
```

uses a `va_list`.

So `vsnprintf()` is used when you are already inside a variadic function and want to forward the variable arguments into a formatter.

---

## Example

```c
#include <stdio.h>
#include <stdarg.h>

int write_message(char *buffer, size_t buffer_size, const char *format, ...)
{
    va_list args;

    va_start(args, format);

    int result = vsnprintf(buffer, buffer_size, format, args);

    va_end(args);

    return result;
}

int main(void)
{
    char buffer[64];

    int n = write_message(buffer, sizeof(buffer), "File %s has %d lines", "main.c", 120);

    printf("buffer: %s\n", buffer);
    printf("result: %d\n", n);

    return 0;
}
```

Output:

```text
buffer: File main.c has 120 lines
result: 25
```

The return value is `25` because that is the number of characters that would have been written, not including the null terminator.

---

## What `vsnprintf()` does internally

Given this call:

```c
vsnprintf(buffer, sizeof(buffer), "File %s has %d lines", args);
```

`vsnprintf()` scans the format string:

```text
File %s has %d lines
```

It copies normal characters directly:

```text
F i l e
```

When it sees a format specifier like:

```c
%s
```

it consumes the next argument from the `va_list` and treats it as a `char *`.

When it sees:

```c
%d
```

it consumes the next argument from the `va_list` and treats it as an `int`.

So conceptually:

```c
%s -> "main.c"
%d -> 120
```

Then it writes the formatted result into the buffer.

---

## Why `vsnprintf()` is safer than `vsprintf()`

There are two similar functions:

```c
vsprintf(buffer, format, args);              // unsafe
vsnprintf(buffer, size, format, args);       // safer
```

`vsprintf()` does not know the size of the destination buffer. If the generated string is too large, it will overflow the buffer.

`vsnprintf()` receives the buffer size:

```c
vsnprintf(buffer, sizeof(buffer), format, args);
```

So it writes at most:

```c
size - 1
```

characters and reserves one byte for the null terminator:

```c
'\0'
```

Example:

```c
char buffer[8];

int n = snprintf(buffer, sizeof(buffer), "Hello, world!");
```

The full string would be:

```text
Hello, world!
```

which is 13 characters.

But the buffer only has space for 8 bytes total, so `snprintf()` writes:

```text
Hello,
```

plus the null terminator.

Memory layout:

```text
H e l l o ,   \0
```

The return value is still:

```text
13
```

because that is the number of characters it wanted to write.

---

## Return value behavior

`vsnprintf()` returns an `int`.

### Case 1: success, no truncation

```c
char buffer[64];

int n = snprintf(buffer, sizeof(buffer), "abc");
```

Result:

```c
buffer = "abc"
n = 3
```

The return value does **not** count the null terminator.

---

### Case 2: success, but truncated

```c
char buffer[4];

int n = snprintf(buffer, sizeof(buffer), "abcdef");
```

The buffer can hold only 4 bytes:

```text
a b c \0
```

Result:

```c
buffer = "abc"
n = 6
```

The return value says:

```text
I needed 6 characters, excluding '\0'
```

Since:

```c
n >= sizeof(buffer)
```

you know truncation occurred.

Correct truncation check:

```c
if (n < 0) {
    // formatting error
} else if ((size_t)n >= sizeof(buffer)) {
    // output was truncated
} else {
    // success, full string written
}
```

This is one of the most important parts of using `vsnprintf()` correctly.

---

## Applying it to your JSON builder

Your append function likely looks like this:

```c
int analysis_append(file_analysis_t *analysis, const char *format, ...)
{
    va_list args;

    va_start(args, format);

    int remaining = ANALYSIS_MAX_SIZE - analysis->position;

    int written = vsnprintf(
        analysis->json + analysis->position,
        remaining,
        format,
        args
    );

    va_end(args);

    if (written < 0) {
        return -1;
    }

    if (written >= remaining) {
        return -1;
    }

    analysis->position += written;

    return 0;
}
```

This line is the key:

```c
vsnprintf(
    analysis->json + analysis->position,
    remaining,
    format,
    args
);
```

Breakdown:

```c
analysis->json + analysis->position
```

means:

```text
write into the buffer starting at the current append position
```

For example, if your buffer already contains:

```json
{
  "file_name": "main.c",
```

then `position` points to the byte after the comma/newline. The next `vsnprintf()` call writes from there.

This:

```c
remaining
```

means:

```text
do not write more than the remaining space in the buffer
```

This:

```c
format
```

is the format string, like:

```c
"  \"line_count\": %u,\n"
```

This:

```c
args
```

contains the actual values for the format specifiers.

---

## Example append sequence

Suppose:

```c
analysis_append(&analysis, "{\n");
```

The buffer becomes:

```json
{
```

Then:

```c
analysis_append(&analysis, "  \"file_name\": \"%s\",\n", "main.c");
```

The format string is:

```c
"  \"file_name\": \"%s\",\n"
```

The `va_list` contains:

```c
"main.c"
```

`vsnprintf()` replaces `%s` with `"main.c"`.

The buffer becomes:

```json
{
  "file_name": "main.c",
```

Then:

```c
analysis_append(&analysis, "  \"line_count\": %u\n", 120);
```

`vsnprintf()` replaces `%u` with:

```text
120
```

The buffer becomes:

```json
{
  "file_name": "main.c",
  "line_count": 120
```

---

## Important: `vsnprintf()` consumes the `va_list`

After passing a `va_list` to `vsnprintf()`, you should treat that `va_list` as used.

This is bad:

```c
va_list args;

va_start(args, format);

int n1 = vsnprintf(buffer1, size1, format, args);
int n2 = vsnprintf(buffer2, size2, format, args); // wrong

va_end(args);
```

The first call may advance the `va_list`. The second call may read garbage or behave incorrectly.

Use `va_copy()` if you need to use it twice:

```c
va_list args;
va_list args_copy;

va_start(args, format);
va_copy(args_copy, args);

int n1 = vsnprintf(buffer1, size1, format, args);
int n2 = vsnprintf(buffer2, size2, format, args_copy);

va_end(args_copy);
va_end(args);
```

This matters in a common pattern where you first calculate the needed size, then allocate a buffer.

---

## Using `vsnprintf()` to calculate required size

You can call `vsnprintf()` with a null buffer and size `0` to calculate how much space is needed:

```c
int needed = vsnprintf(NULL, 0, format, args);
```

But because `vsnprintf()` consumes `args`, you need `va_copy()`:

```c
char *format_alloc(const char *format, ...)
{
    va_list args;
    va_list args_copy;

    va_start(args, format);
    va_copy(args_copy, args);

    int needed = vsnprintf(NULL, 0, format, args);

    if (needed < 0) {
        va_end(args_copy);
        va_end(args);
        return NULL;
    }

    char *buffer = malloc((size_t)needed + 1);

    if (buffer == NULL) {
        va_end(args_copy);
        va_end(args);
        return NULL;
    }

    vsnprintf(buffer, (size_t)needed + 1, format, args_copy);

    va_end(args_copy);
    va_end(args);

    return buffer;
}
```

The `+ 1` is for the null terminator.

---

## Most important rules

Use `vsnprintf()` when writing a formatting wrapper around a variadic function.

Always pass the real remaining buffer size.

Always check:

```c
written < 0
```

for formatting error.

Always check:

```c
(size_t)written >= remaining
```

for truncation.

Always call:

```c
va_end(args);
```

after `va_start()`.

Use `va_copy()` if the same argument list must be used more than once.

For your JSON builder, `vsnprintf()` is the core function that lets you safely append formatted text into a fixed-size C buffer without overrunning it.