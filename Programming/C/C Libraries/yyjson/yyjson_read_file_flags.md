Related to [[yyjson]], [[JSON]], [[C]]
#yyjson #yyjson_read_file #C 


## List of Flags

The `flg` parameter of `yyjson_read_file()` has type `yyjson_read_flag`, which is a `uint32_t` bitmask:

```c
yyjson_doc *yyjson_read_file(
    const char *path,
    yyjson_read_flag flg,
    const yyjson_alc *alc,
    yyjson_read_err *err
);
```

You can pass one flag or combine multiple flags using bitwise OR (`|`). ([IbiReme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

## Default parsing

### `YYJSON_READ_NOFLAG`

Equivalent to `0`:

```c
yyjson_doc *doc = yyjson_read_file(
    "data.json",
    YYJSON_READ_NOFLAG,
    NULL,
    NULL
);
```

This performs strict, standards-compliant JSON parsing. It rejects comments, trailing commas, BOMs, invalid UTF-8, `NaN`, and `Infinity`. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

You will usually use either:

```c
YYJSON_READ_NOFLAG
```

or simply:

```c
0
```

## Parsing-control flags

### `YYJSON_READ_INSITU`

Allows the parser to modify and reuse the input buffer for storing strings, potentially reducing copying.

```c
YYJSON_READ_INSITU
```

This flag is primarily relevant when parsing a mutable buffer with `yyjson_read_opts()`. The buffer must remain alive until the document is freed and must include `YYJSON_PADDING_SIZE` bytes of padding. For ordinary `yyjson_read_file()` usage, you generally do not need to select it yourself. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

### `YYJSON_READ_STOP_WHEN_DONE`

Stops after parsing the first complete JSON value instead of reporting an error for additional content.

For example, this input normally fails:

```text
{"id": 1} {"id": 2}
```

With this flag, the parser stops after:

```json
{"id": 1}
```

It is useful for JSON or streams containing multiple JSON documents. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

## Nonstandard JSON syntax flags

### `YYJSON_READ_ALLOW_TRAILING_COMMAS`

Allows trailing commas:

```json
{
    "name": "Harrison",
    "age": 25,
}
```

It also permits them in arrays:

```json
[1, 2, 3,]
```

### `YYJSON_READ_ALLOW_COMMENTS`

Allows C-style comments:

```json
{
    // Single-line comment
    "name": "Harrison",

    /* Multi-line comment */
    "age": 25
}
```

### `YYJSON_READ_ALLOW_INF_AND_NAN`

Allows nonstandard floating-point values:

```json
{
    "maximum": Infinity,
    "minimum": -Infinity,
    "result": NaN
}
```

It also accepts case-insensitive variants such as `inf`, `Inf`, and `nan`. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

### `YYJSON_READ_ALLOW_BOM`

Allows and skips a UTF-8 byte-order mark at the beginning of the file.

Strict JSON parsing normally rejects a BOM.

### `YYJSON_READ_ALLOW_EXT_NUMBER`

Allows extended number formats, including:

```json
{
    "hex": 0x7B,
    "leading_decimal": .123,
    "trailing_decimal": 123.,
    "positive": +123
}
```

### `YYJSON_READ_ALLOW_EXT_ESCAPE`

Allows extended string escape sequences, including:

```text
\a
\e
\v
\'
\?
\0
\x7B
```

It also permits line continuations and some otherwise unsupported escape characters.

### `YYJSON_READ_ALLOW_EXT_WHITESPACE`

Allows additional whitespace characters, including vertical tabs, form feeds, Unicode line and paragraph separators, nonbreaking spaces, and other Unicode separator characters.

### `YYJSON_READ_ALLOW_SINGLE_QUOTED_STR`

Allows single-quoted strings:

```json5
{
    "name": 'Harrison'
}
```

### `YYJSON_READ_ALLOW_UNQUOTED_KEY`

Allows object keys without quotes:

```json5
{
    name: "Harrison",
    age: 25
}
```

These options all permit syntax outside strict RFC 8259 JSON. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

## Number-handling flags

### `YYJSON_READ_NUMBER_AS_RAW`

Stores every JSON number as raw text rather than converting it into an integer or floating-point representation.

For this JSON:

```json
{
    "value": 123.456
}
```

`value` is stored as a raw value containing:

```text
123.456
```

You retrieve it using:

```c
const char *raw = yyjson_get_raw(value);
```

This is useful when you need custom number parsing or exact decimal text.

### `YYJSON_READ_BIGNUM_AS_RAW`

Only stores numbers as raw text when they cannot be represented safely as:

- `int64_t`
    
- `uint64_t`
    
- A finite `double`
    

Normal-sized numbers are still parsed normally.

`YYJSON_READ_NUMBER_AS_RAW` overrides `YYJSON_READ_BIGNUM_AS_RAW` when both are specified. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

## Unicode handling

### `YYJSON_READ_ALLOW_INVALID_UNICODE`

Allows invalid UTF-8 bytes inside parsed string values.

```c
YYJSON_READ_ALLOW_INVALID_UNICODE
```

This should generally be avoided in a validation pipeline because resulting strings might contain malformed text and require careful security handling. Invalid escape sequences are still rejected. ([GitHub](https://github.com/ibireme/yyjson/blob/master/doc/API.md "yyjson/doc/API.md at master · ibireme/yyjson · GitHub"))

## JSON5 convenience flag

### `YYJSON_READ_JSON5`

Enables the supported JSON5-style syntax as one combined flag:

```c
yyjson_doc *doc = yyjson_read_file(
    "config.json5",
    YYJSON_READ_JSON5,
    NULL,
    NULL
);
```

Internally, it combines:

```c
YYJSON_READ_ALLOW_TRAILING_COMMAS |
YYJSON_READ_ALLOW_COMMENTS |
YYJSON_READ_ALLOW_INF_AND_NAN |
YYJSON_READ_ALLOW_EXT_NUMBER |
YYJSON_READ_ALLOW_EXT_ESCAPE |
YYJSON_READ_ALLOW_EXT_WHITESPACE |
YYJSON_READ_ALLOW_SINGLE_QUOTED_STR |
YYJSON_READ_ALLOW_UNQUOTED_KEY
```

It does not automatically enable invalid Unicode, raw-number handling, BOM support, in-situ parsing, or stop-when-done behavior. ([IbiReme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

## Combining flags

For example, to accept comments and trailing commas:

```c
yyjson_read_flag flags =
    YYJSON_READ_ALLOW_COMMENTS |
    YYJSON_READ_ALLOW_TRAILING_COMMAS;

yyjson_read_err err;

yyjson_doc *doc = yyjson_read_file(
    "data.json",
    flags,
    NULL,
    &err
);

if (doc == NULL) {
    fprintf(
        stderr,
        "JSON error: %s at byte %zu\n",
        err.msg,
        err.pos
    );
    return;
}
```

For your JSON validation project, strict parsing is normally the appropriate default:

```c
YYJSON_READ_NOFLAG
```

Only enable individual permissive flags when the input format explicitly allows those extensions.


