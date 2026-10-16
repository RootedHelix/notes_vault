Related to [[C]]
#fgets

## Personal Notes

**Function Signature**

```c
char* fgets(char *str, int n, FILE *stream);
```


### **Parameter Definitions**

 `char *str` A pointer to the destination buffer where the characters read from the file will be                           stored.


**Parameter 2: `int n`**

The maximum number of characters that can be stored in the buffer, including the null terminator.

`fgets()` will read at most `n - 1` characters and then append `'\0'`.

Example:

```
char buffer[10];fgets(buffer, 10, file);
```

Maximum characters read:

```
9 characters + '\0'
```

If the file contains:

```
HelloWorld123
```

The buffer becomes:

```
HelloWorl\0
```

and the remaining characters stay in the stream for the next read.

**Parameter 3: `FILE *stream`**

The input stream to read from.

Common examples:

```
FILE *file = fopen("data.txt", "r");fgets(buffer, sizeof(buffer), file);
```

or reading from standard input:

```
fgets(buffer, sizeof(buffer), stdin);
```

`stdin` is also a `FILE *`


### **Key Features**



## AI Response

`fgets()` reads a line of text from a file stream into a character buffer.


### Example of Function use

```c
#include <stdio.h>

int main(void)
{
    char buffer[128];

    printf("Enter text: ");

    if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
        printf("You typed: %s", buffer);
    }

    return 0;
}
```

`fgets()` reads from `stream`, such as:

```c
stdin       // keyboard input
FILE *fp    // file opened with fopen()
```

It stores the result into `str`, but reads at most `n - 1` characters. The last byte is reserved for the null terminator `'\0'`.

So this:

```c
char buffer[128];

fgets(buffer, sizeof(buffer), stdin);
```

means:

> Read up to 127 characters from standard input into `buffer`, then append `'\0'`.

Important behavior: `fgets()` stops when one of these happens:

1. It reads a newline character `'\n'`
    
2. It reaches `n - 1` characters
    
3. It reaches end-of-file
    
4. An error occurs
    

Unlike many input functions, `fgets()` usually **keeps the newline** if there is room for it.

Example input:

```text
hello
```

The buffer may contain:

```c
"hello\n\0"
```

That means this comparison would fail:

```c
if (strcmp(buffer, "hello") == 0) {
    // usually false, because buffer is "hello\n"
}
```

You often remove the newline like this:

```c
#include <string.h>

buffer[strcspn(buffer, "\n")] = '\0';
```

Full example:

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char name[64];

    printf("Enter your name: ");

    if (fgets(name, sizeof(name), stdin) == NULL) {
        printf("Input error or EOF\n");
        return 1;
    }

    name[strcspn(name, "\n")] = '\0';

    printf("Hello, %s\n", name);

    return 0;
}
```

The return value matters:

```c
char *result = fgets(buffer, sizeof(buffer), stdin);
```

If successful, `fgets()` returns `buffer`.

If it fails or reaches EOF before reading anything, it returns `NULL`.

So the safe pattern is:

```c
if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
    // use buffer
}
```

For your JSON-in-C context, `fgets()` is useful when reading input line-by-line from a file:

```c
FILE *fp = fopen("data.txt", "r");
char line[256];

if (fp == NULL) {
    return 1;
}

while (fgets(line, sizeof(line), fp) != NULL) {
    printf("Line: %s", line);
}

fclose(fp);
```

Key point: **`fgets()` is a bounded input function.** It is much safer than old functions like `gets()` because it knows the size of your buffer and avoids writing past the end, assuming you pass the correct buffer size.