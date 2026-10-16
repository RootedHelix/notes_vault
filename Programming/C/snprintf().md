Related to [[C]], [[<string.h>]]
#snprintf #C 

## Personal Notes

> `snprintf()` writes formatted text into a fixed-size character buffer.

It is the safer version of `sprintf()` because you tell it the size of the destination buffer.


**Function signature**:

```c
int snprintf(char *str, size_t size, const char *format, ...);
```


## AI Response

### Basic idea

```c
char buffer[32];

snprintf(buffer, sizeof(buffer), "Age: %d", 25);
```

After this, `buffer` contains:

```text
"Age: 25"
```

	### Parameters

```c
snprintf(buffer, sizeof(buffer), "x = %d", value);
```

|Parameter|Meaning|
|---|---|
|`buffer`|Destination array to write into|
|`sizeof(buffer)`|Maximum number of bytes available|
|`"x = %d"`|Format string|
|`value`|Value inserted into `%d`|

## Why it is safer than `sprintf()`

Unsafe:

```c
char buffer[8];

sprintf(buffer, "hello world"); // dangerous: writes past buffer
```

Safer:

```c
char buffer[8];

snprintf(buffer, sizeof(buffer), "hello world");
```

This prevents writing more than the buffer can hold.

The buffer would contain:

```text
"hello w"
```

Because the 8-byte buffer must reserve one byte for the null terminator:

```c
'h' 'e' 'l' 'l' 'o' ' ' 'w' '\0'
```

So `snprintf()` writes at most:

```c
size - 1
```

visible characters, then appends:

```c
'\0'
```

## Important: `snprintf()` can truncate

If the formatted output is too large, `snprintf()` cuts it off so it fits.

Example:

```c
char buffer[10];

snprintf(buffer, sizeof(buffer), "Number: %d", 12345);
```

The full string would be:

```text
"Number: 12345"
```

That is too large for 10 bytes, so `buffer` may contain:

```text
"Number: 1"
```

plus the null terminator.

## Return value

This part is very important.

`snprintf()` returns the number of characters it **wanted** to write, not including the null terminator.

Example:

```c
char buffer[10];

int needed = snprintf(buffer, sizeof(buffer), "Number: %d", 12345);

printf("needed = %d\n", needed);
printf("buffer = %s\n", buffer);
```

Output:

```text
needed = 13
buffer = Number: 1
```

Why `13`?

```text
Number: 12345
```

has 13 characters.

The buffer only had room for 9 visible characters plus `'\0'`.

So you can detect truncation like this:

```c
if (needed < 0) {
    // encoding or formatting error
} else if ((size_t)needed >= sizeof(buffer)) {
    // output was truncated
}
```

## Common use

Building a small string safely:

```c
char msg[64];

int sensor_id = 7;
float voltage = 3.72f;

snprintf(msg, sizeof(msg), "sensor=%d voltage=%.2f", sensor_id, voltage);
```

Result:

```text
sensor=7 voltage=3.72
```

## Difference between `sprintf()` and `snprintf()`

|Function|Safer?|Why|
|---|--:|---|
|`sprintf()`|No|Does not know buffer size|
|`snprintf()`|Yes|Limits writes to the provided size|

In modern C, prefer:

```c
snprintf()
```

over:

```c
sprintf()
```

almost always.

## Key summary

`snprintf()`:

- Formats text like `printf()`
    
- Writes into a character buffer
    
- Takes the buffer size
    
- Writes at most `size - 1` characters
    
- Adds `'\0'` if `size > 0`
    
- Truncates if the output is too long
    
- Returns how many characters would have been written
    

The safe pattern is:

```c
char buffer[128];

int n = snprintf(buffer, sizeof(buffer), "name=%s age=%d", name, age);

if (n < 0) {
    // formatting error
} else if ((size_t)n >= sizeof(buffer)) {
    // truncated
}
```