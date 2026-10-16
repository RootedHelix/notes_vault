Related to [[yyjson]], [[JSON]], [[C]]
#yyjson #yyjson_read_err
## Personal Response

**Definition**

`yyjson_read_err` is a **structure used by yyjson to return detailed parsing-error information**. It is not a function.

**Struct Definition**

```c
typedef struct yyjson_read_err {
    yyjson_read_code code;
    const char *msg;
    size_t pos;
} yyjson_read_err;
```

Its fields are:

```c
err.code  // Numeric/error-code category
err.msg   // Human-readable error message
err.pos   // Byte offset where the error occurred
```

`msg` points to a constant string owned by yyjson, so you must **not free it**. `pos` is a byte position, not necessarily a character index for multibyte UTF-8 text. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

### Typical use

```c
#include <stdio.h>
#include <yyjson.h>

yyjson_doc *read_json_file(const char *path)
{
    yyjson_read_err err = {0};

    yyjson_doc *doc = yyjson_read_file(
        path,
        0,      // No special parsing flags
        NULL,   // Use the default allocator
        &err    // Receive error information
    );

    if (doc == NULL) {
        fprintf(
            stderr,
            "JSON read error: code=%u, message=%s, byte=%zu\n",
            err.code,
            err.msg != NULL ? err.msg : "unknown error",
            err.pos
        );

        return NULL;
    }

    return doc;
}
```


## AI Response


The important part is:

```c
&err
```

You pass the address of your local `yyjson_read_err` structure. yyjson writes the error information into it.

The `yyjson_read_file()` signature is effectively:

```c
yyjson_doc *yyjson_read_file(
    const char *path,
    yyjson_read_flag flags,
    const yyjson_alc *allocator,
    yyjson_read_err *err
);
```

The final parameter is optional. Passing `NULL` means you do not want detailed error information. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

## What the error might contain

For invalid JSON such as:

```json
{
    "name": "Harrison",
}
```

strict JSON parsing may fail because of the trailing comma. You might receive information similar to:

```text
code=7
message=unexpected trailing comma
byte=25
```

The exact message text can vary, but the code identifies categories such as:

```c
YYJSON_READ_SUCCESS
YYJSON_READ_ERROR_INVALID_PARAMETER
YYJSON_READ_ERROR_MEMORY_ALLOCATION
YYJSON_READ_ERROR_EMPTY_CONTENT
YYJSON_READ_ERROR_UNEXPECTED_CONTENT
YYJSON_READ_ERROR_UNEXPECTED_END
YYJSON_READ_ERROR_UNEXPECTED_CHARACTER
YYJSON_READ_ERROR_JSON_STRUCTURE
YYJSON_READ_ERROR_INVALID_NUMBER
YYJSON_READ_ERROR_INVALID_STRING
YYJSON_READ_ERROR_LITERAL
YYJSON_READ_ERROR_FILE_OPEN
YYJSON_READ_ERROR_FILE_READ
```

These represent problems such as malformed structure, an invalid number, an incomplete document, or failure to open the file. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

## Check the document pointer first

Use this:

```c
if (doc == NULL) {
    // Parsing or file-reading failed
}
```

Do not rely only on:

```c
if (err.pos == 0)
```

An error can legitimately occur at byte zero. The reliable success test is whether the returned `yyjson_doc *` is non-`NULL`, or whether `err.code` equals `YYJSON_READ_SUCCESS`.

When successful, remember to release the document:

```c
yyjson_doc_free(doc);
```