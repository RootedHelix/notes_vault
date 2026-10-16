Related to [[yyjson]], [[JSON]], [[C]]
#yyjson #yyjson_read_file #C 

## Personal Response

`yyjson_read_file()` opens a JSON file, reads its contents, parses the JSON, and returns a new immutable `yyjson_doc`.

**Function Signature**

```c
yyjson_doc *yyjson_read_file(
    const char *path,
    yyjson_read_flag flg,
    const yyjson_alc *alc,
    yyjson_read_err *err
);
```

### Parameters

#### `path`

```c
const char *path
```

The null-terminated path to the JSON file:

```c
"data.json"
```

or:

```c
"/Users/sam/project/data.json"
```

The path may be relative or absolute. The function returns `NULL` when the path is invalid or the file cannot be opened. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

#### `flg`

```c
yyjson_read_flag flg
```

Controls how yyjson parses the file.

Using `0` means standard strict JSON parsing:

```c
yyjson_read_file("data.json", 0, NULL, NULL);
```

Flags can be combined using the bitwise OR operator:

```c
yyjson_read_file(
    "data.json",
    YYJSON_READ_ALLOW_COMMENTS |
    YYJSON_READ_ALLOW_TRAILING_COMMAS,
    NULL,
    NULL
);
```

That example allows JSON extensions such as comments and trailing commas.

**List of flags**

[[yyjson_read_files_flags]]


#### `alc`

```c
const yyjson_alc *alc
```

Specifies the allocator yyjson should use.

For normal usage, pass `NULL`:

```c
NULL
```

This tells yyjson to use the default C allocation functions. A custom `yyjson_alc` is useful when you need an arena, pool allocator, embedded allocation policy, or custom memory tracking. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

#### `err`

```c
yyjson_read_err *err
```

Receives detailed error information. Pass `NULL` when you only need to know whether parsing succeeded.


## AI Response

It effectively combines these operations:

```text
open file
   ↓
read file contents
   ↓
parse JSON text
   ↓
allocate yyjson_doc
   ↓
return parsed document
```

You do **not** need to call `fopen()`, `fread()`, or `fclose()` yourself when using this function.

## Basic example

```c
#include <stdio.h>
#include <yyjson.h>

int main(void) {
    yyjson_doc *doc = yyjson_read_file(
        "config.json",
        0,
        NULL,
        NULL
    );

    if (doc == NULL) {
        fprintf(stderr, "Failed to read or parse config.json\n");
        return 1;
    }

    yyjson_val *root = yyjson_doc_get_root(doc);

    /* Use the parsed JSON here. */

    yyjson_doc_free(doc);
    return 0;
}
```

The returned document owns all parsed JSON values and strings, so it must eventually be released using `yyjson_doc_free()`. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))



For production code, supplying an error structure is preferable:

```c
yyjson_read_err err;

yyjson_doc *doc = yyjson_read_file(
    "config.json",
    0,
    NULL,
    &err
);

if (doc == NULL) {
    fprintf(
        stderr,
        "JSON error: %s, code: %u, byte position: %zu\n",
        err.msg,
        err.code,
        err.pos
    );

    return 1;
}
```

The error can represent either a file-reading failure or invalid JSON syntax. `err.pos` reports the byte position associated with a JSON parsing error. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

## Reading values from the result

Given this file:

```json
{
  "name": "Alice",
  "age": 30
}
```

You could parse it like this:

```c
yyjson_read_err err;

yyjson_doc *doc = yyjson_read_file(
    "person.json",
    0,
    NULL,
    &err
);

if (doc == NULL) {
    fprintf(stderr, "Error: %s\n", err.msg);
    return 1;
}

yyjson_val *root = yyjson_doc_get_root(doc);

if (!yyjson_is_obj(root)) {
    fprintf(stderr, "Expected the root value to be an object\n");
    yyjson_doc_free(doc);
    return 1;
}

yyjson_val *name_val = yyjson_obj_get(root, "name");
yyjson_val *age_val = yyjson_obj_get(root, "age");

if (yyjson_is_str(name_val)) {
    printf("Name: %s\n", yyjson_get_str(name_val));
}

if (yyjson_is_int(age_val)) {
    printf("Age: %lld\n", (long long)yyjson_get_sint(age_val));
}

yyjson_doc_free(doc);
```

## Return value

On success:

```c
yyjson_doc *doc;
```

On failure:

```c
NULL
```

Therefore, always check the result:

```c
yyjson_doc *doc = yyjson_read_file(path, 0, NULL, &err);

if (doc == NULL) {
    /* Handle error. */
}
```

The core distinction from `yyjson_read()` is:

```c
yyjson_read(...)
```

parses JSON already stored in memory, while:

```c
yyjson_read_file(...)
```

loads and parses JSON directly from a file path. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))
