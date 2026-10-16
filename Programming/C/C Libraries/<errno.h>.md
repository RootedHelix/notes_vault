Type: **int** 

Description: - is an int that holds a thread-local error code set by the most recent failing library                           or system call.
## errno

#### AI Response 

`errno` is a C error-reporting mechanism used by many standard library and operating-system functions.

It is defined in:

```c
#include <errno.h>
```

Conceptually, `errno` stores an integer error code describing the last error that occurred in certain library/system calls.

Example:

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>

int main(void)
{
    FILE *fp = fopen("missing.txt", "r");

    if (fp == NULL) {
        printf("errno = %d\n", errno);
        printf("error = %s\n", strerror(errno));
        perror("fopen");
    }

    return 0;
}
```

Possible output:

```text
errno = 2
error = No such file or directory
fopen: No such file or directory
```

Here, `fopen()` failed, so it set `errno` to an error code such as `ENOENT`, meaning “no such file or directory.”

You can compare `errno` against named constants:

```c
if (errno == ENOENT) {
    printf("File does not exist\n");
}
```

Common `errno` values include:

```c
ENOENT   // No such file or directory
EACCES   // Permission denied
EINVAL   // Invalid argument
ENOMEM   // Not enough memory
ERANGE   // Result out of range
```

Important: `errno` is called a **macro**, not necessarily a normal global variable. The C standard defines it as a macro that expands to a **modifiable lvalue** of type `int`.

So you can do this:

```c
errno = 0;
```

But internally it may expand to something more complex, especially on modern systems where each thread needs its own separate `errno`.

For example, conceptually:

```c
#define errno (*__errno_location())
```

That means every thread can have its own error state.

The most important rule:

```c
errno is only meaningful after a function reports failure.
```

Do **not** check `errno` by itself.

Bad:

```c
fopen("file.txt", "r");

if (errno != 0) {
    // wrong pattern
}
```

Better:

```c
FILE *fp = fopen("file.txt", "r");

if (fp == NULL) {
    perror("fopen");
}
```

Also, successful functions are **not required to reset `errno` to 0**. So `errno` may still contain an old error value even after a later function succeeds.

That is why this can be misleading:

```c
errno = 0;

printf("hello\n");

if (errno != 0) {
    // not a reliable way to check printf unless printf itself indicated failure
}
```

For some functions, you intentionally clear `errno` before calling them. `strtol()` is a common example:

```c
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>

int main(void)
{
    char *end;
    errno = 0;

    long value = strtol("999999999999999999999999", &end, 10);

    if (errno == ERANGE) {
        printf("Number out of range\n");
    } else {
        printf("value = %ld\n", value);
    }

    return 0;
}
```

Why set `errno = 0` first? Because `strtol()` returns a numeric value, and some valid return values can look like failure values. Clearing `errno` lets you detect whether an overflow/underflow happened.

So, in practical terms:

```c
errno
```

means:

> “An integer error code set by certain C/library/system functions after they fail.”

You usually use it with:

```c
perror("context");
```

or:

```c
strerror(errno);
```

Example:

```c
if (some_function_failed) {
    fprintf(stderr, "Error: %s\n", strerror(errno));
}
```