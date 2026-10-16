Related to [[C]], 
#fputs #file_functions #C 

## Personal Notes

`fputs()` writes a C string to a file stream.

**Function signature**:

```c
int fputs(const char *str, FILE *stream);
```

Return behavior:

```c
if (fputs("data\n", fp) == EOF) {
    // failure
}
```

On success, `fputs()` returns a non-negative value. On **failure**, it returns `EOF`. What is [[EOF]]

So this is standard:

```c
int write_line(FILE *fp, const char *line)
{
    if (fp == NULL || line == NULL) {
        return -1;
    }

    if (fputs(line, fp) == EOF) {
        return -1;
    }

    if (fputc('\n', fp) == EOF) {
        return -1;
    }

    return 0;
}
```

Use `fputs()` when you want to write a plain string to a file. Use `fprintf()` when you need formatting, and use `fwrite()` when writing binary data or a known number of bytes.


## AI Response


Example:

```c
FILE *fp = fopen("output.txt", "w");

if (fp == NULL) {
    return -1;
}

if (fputs("Hello, world\n", fp) == EOF) {
    // write failed
}

fclose(fp);
```

`fputs()` takes the characters from `str` and writes them into `stream`.

Important detail: unlike `puts()`, `fputs()` does **not** automatically add a newline.

```c
fputs("hello", fp);
fputs("world", fp);
```

This writes:

```text
helloworld
```

If you want a newline, include it yourself:

```c
fputs("hello\n", fp);
fputs("world\n", fp);
```

This writes:

```text
hello
world
```