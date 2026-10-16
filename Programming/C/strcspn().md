Related to [[C]], [[<string.h>]]
#strcspn

## Personal Notes

> `strcspn()` means **string complement span**.
> It tells you:

> “How many characters can I move through this string before I hit one of these forbidden characters?”


**Function signature**:

```c
size_t strcspn(const char *s, const char *reject);
```



## AI Response

### Basic example

```c
#include <string.h>
#include <stdio.h>

int main(void) {
    char text[] = "hello\n";

    size_t index = strcspn(text, "\n");

    printf("%zu\n", index);
}
```

Output:

```text
5
```

Why? Because:

```text
h e l l o \n
0 1 2 3 4 5
```

The first `'\n'` is at index `5`, so `strcspn()` returns `5`.

## Common use with `fgets()`

This is the classic pattern:

```c
char buffer[20];

if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
    buffer[strcspn(buffer, "\n")] = '\0';
}
```

This removes the newline from `fgets()` **if one exists**.

Example input:

```text
hello
```

After `fgets()`:

```c
"hello\n"
```

After this line:

```c
buffer[strcspn(buffer, "\n")] = '\0';
```

The string becomes:

```c
"hello"
```

## Why this works

If the newline exists:

```c
buffer[strcspn(buffer, "\n")] = '\0';
```

becomes:

```c
buffer[5] = '\0';
```

So the newline is replaced with the null terminator.

If the newline does **not** exist, `strcspn()` returns the index of the existing null terminator.

Example:

```c
char buffer[6] = "hello";
```

There is no newline. Internally:

```c
'h' 'e' 'l' 'l' 'o' '\0'
```

`strcspn(buffer, "\n")` returns `5`.

Then:

```c
buffer[5] = '\0';
```

That writes `'\0'` where there was already `'\0'`. No problem.

## Is `strcspn()` safe?

**It is safe when used on valid null-terminated strings.**

This is safe:

```c
char buffer[20];

if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
    buffer[strcspn(buffer, "\n")] = '\0';
}
```

Because `fgets()` gives you a null-terminated string as long as it succeeds.

But `strcspn()` is **not safe for raw buffers that are not guaranteed to contain `'\0'`**.

Unsafe example:

```c
char buffer[5] = {'h', 'e', 'l', 'l', 'o'};

size_t n = strcspn(buffer, "\n"); // unsafe
```

There is no null terminator, so `strcspn()` may keep reading past the end of the array looking for either `'\n'` or `'\0'`. That is undefined behavior.

## Important point

`strcspn()` itself does **not modify** the string.

This only calculates an index:

```c
size_t i = strcspn(buffer, "\n");
```

This modifies the string:

```c
buffer[i] = '\0';
```

So with `fgets()`, the full pattern is:

```c
if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
    buffer[strcspn(buffer, "\n")] = '\0';
}
```

That means:

> Read input safely, then replace the newline with `'\0'` if the newline is present.