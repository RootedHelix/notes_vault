Related to [[C]], [[<stdlib>]]
#stroul

## Personal Notes

- String to unsigned long function

## AI Response

`strtoul()` converts a string into an **unsigned long integer**.

It is declared in:

```c
#include <stdlib.h>
```

Prototype:

```c
unsigned long strtoul(const char *nptr, char **endptr, int base);
```

## What it does

`strtoul()` reads characters from a C string and interprets them as a number.

Example:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    const char *s = "12345";
    unsigned long value = strtoul(s, NULL, 10);

    printf("%lu\n", value);

    return 0;
}
```

Output:

```text
12345
```

## Parameters

```c
strtoul(nptr, endptr, base)
```

### `nptr`

The string you want to convert.

```c
"1234"
"0xff"
"0755"
```

### `endptr`

A pointer that tells you **where parsing stopped**.

Example:

```c
char *end;
unsigned long value = strtoul("123abc", &end, 10);

printf("value = %lu\n", value);
printf("stopped at: %s\n", end);
```

Output:

```text
value = 123
stopped at: abc
```

So `endptr` is useful for detecting invalid trailing characters.

### `base`

The numeric base to use.

Common values:

```c
10  // decimal
16  // hexadecimal
8   // octal
2   // binary, supported by many systems but not always portable C
0   // auto-detect base
```

With `base = 0`, C auto-detects:

```c
"123"    // decimal
"077"    // octal
"0xff"   // hexadecimal
```

Example:

```c
unsigned long a = strtoul("255", NULL, 10);    // 255
unsigned long b = strtoul("ff", NULL, 16);     // 255
unsigned long c = strtoul("0xff", NULL, 0);    // 255
```

## Why use `strtoul()` instead of `atoi()`?

`strtoul()` is safer and more controllable because it lets you detect errors.

Bad:

```c
int x = atoi("abc");  // returns 0, but you cannot tell if input was invalid
```

Better:

```c
char *end;
unsigned long x = strtoul("abc", &end, 10);

if (end == "abc") {
    // no digits were parsed
}
```

Usually you compare against the original pointer:

```c
const char *s = "abc";
char *end;

unsigned long x = strtoul(s, &end, 10);

if (end == s) {
    printf("No valid number found\n");
}
```

## Overflow handling

If the number is too large for `unsigned long`, `strtoul()` returns `ULONG_MAX` and sets `errno` to `ERANGE`.

Example:

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <limits.h>

int main(void) {
    const char *s = "999999999999999999999999999999999";
    char *end;

    errno = 0;
    unsigned long value = strtoul(s, &end, 10);

    if (errno == ERANGE) {
        printf("Overflow occurred\n");
    }

    return 0;
}
```

## Important note about negative strings

Even though `strtoul()` returns an **unsigned** value, it may accept a leading minus sign:

```c
unsigned long x = strtoul("-1", NULL, 10);
```

That result wraps according to unsigned conversion rules, often becoming:

```text
ULONG_MAX
```

So if you do not want negative input, check for `'-'` manually before calling it.

## Practical validation pattern

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <limits.h>

int parse_ulong(const char *s, unsigned long *out) {
    char *end;

    if (s == NULL || *s == '\0') {
        return 0;
    }

    if (*s == '-') {
        return 0;
    }

    errno = 0;
    unsigned long value = strtoul(s, &end, 10);

    if (end == s) {
        return 0; // no digits parsed
    }

    if (*end != '\0') {
        return 0; // extra invalid characters
    }

    if (errno == ERANGE) {
        return 0; // overflow
    }

    *out = value;
    return 1;
}
```

Usage:

```c
unsigned long n;

if (parse_ulong("1234", &n)) {
    printf("Valid: %lu\n", n);
} else {
    printf("Invalid number\n");
}
```

In simple terms: `strtoul()` is the standard C function for converting a string like `"123"` or `"0xff"` into an `unsigned long`, while giving you tools to detect where parsing stopped and whether overflow occurred.