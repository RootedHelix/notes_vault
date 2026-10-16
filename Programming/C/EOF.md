Related to [[C]]
#eof #C 

## Personal Notes 

`EOF` means **End Of File**.


In C, `EOF` is a macro defined in `<stdio.h>` that represents a special value used by input/output functions to signal:

1. the end of input was reached, or
    
2. an error occurred.
    

It is usually defined as `-1`, but you should not rely on the exact value. Use the name `EOF`.

```c
#include <stdio.h>
```


## AI Response

## Example with `fgetc()`

```c
int ch;

while ((ch = fgetc(fp)) != EOF) {
    putchar(ch);
}
```

This reads characters from a file until `fgetc()` returns `EOF`.

Important: `ch` is an `int`, not a `char`.

That is because `fgetc()` needs to return either:

- a valid character value, or
    
- the special `EOF` value
    

So this is correct:

```c
int ch = fgetc(fp);
```

This is weaker:

```c
char ch = fgetc(fp); // bad pattern
```

A `char` may not be able to represent `EOF` distinctly from real byte values.

## EOF does not always mean “no error”

After a read loop ends, you may need to check whether it ended because of normal end-of-file or because of an error.

```c
int ch;

while ((ch = fgetc(fp)) != EOF) {
    putchar(ch);
}

if (ferror(fp)) {
    printf("Read error occurred\n");
} else if (feof(fp)) {
    printf("Reached end of file\n");
}
```

## With `fputs()`

For `fputs()`, `EOF` means the write failed:

```c
if (fputs("hello\n", fp) == EOF) {
    // write failed
}
```

So in that context, `EOF` is being used as a generic I/O failure signal, not literally “end of file.”

## Simple definition

You can think of it as:

```c
EOF = special int value meaning "no more input" or "I/O failure"
```

For input functions like `fgetc()`, it usually means end of file or read error.

For output functions like `fputs()`, it means write error.
