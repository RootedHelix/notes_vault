Related to [[yyjson]], [[JSON]], [[C]]
#yyjson #yyjson_doc


## Personal Response

**Definition**
yyjson_doc represents an entire **parsed, immutable JSON document**. It owns the memory used by the parsed JSON values and strings and provides access to the document’s root value. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

**Struct Definition**

```c
struct yyjson_doc {
    yyjson_val *root;
    yyjson_alc alc;
    size_t dat_read;
    size_t val_read;
    char *str_pool;
};
```

Their roles are:

- `root`: the top-level JSON value.
    
- `alc`: the allocator used for the document.
    
- `dat_read`: number of input bytes consumed.
    
- `val_read`: number of JSON values parsed.
    
- `str_pool`: storage used for parsed strings. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))
    

## AI Response


Conceptually:

```text
yyjson_doc
│
├── root JSON value
│   ├── objects
│   ├── arrays
│   ├── strings
│   ├── numbers
│   └── booleans/null
│
├── parsed string storage
├── parsed value storage
└── allocator/lifetime information
```

## Typical usage

```c
const char *json =
    "{"
    "  \"name\": \"Alice\","
    "  \"age\": 30"
    "}";

yyjson_doc *doc = yyjson_read(json, strlen(json), 0);

if (doc == NULL) {
    printf("Failed to parse JSON\n");
    return;
}

yyjson_val *root = yyjson_doc_get_root(doc);

yyjson_val *name = yyjson_obj_get(root, "name");
yyjson_val *age  = yyjson_obj_get(root, "age");

printf("name: %s\n", yyjson_get_str(name));
printf("age: %lld\n", (long long)yyjson_get_sint(age));

yyjson_doc_free(doc);
```

The relationship is:

```c
yyjson_doc *doc;   // owns the complete parsed document
yyjson_val *root;  // points to the root value inside doc
yyjson_val *name;  // points to another value inside doc
```


You normally should not access these fields directly. Use functions such as:

```c
yyjson_doc_get_root(doc);
yyjson_doc_get_read_size(doc);
yyjson_doc_get_val_count(doc);
```

## The document owns its values

A crucial rule is that `yyjson_val *` pointers obtained from the document are only valid while the document exists.

```c
yyjson_val *root = yyjson_doc_get_root(doc);

yyjson_doc_free(doc);

/* root is now invalid */
```

Do not do this:

```c
const char *name = yyjson_get_str(
    yyjson_obj_get(yyjson_doc_get_root(doc), "name")
);

yyjson_doc_free(doc);

printf("%s\n", name);  // invalid: name pointed into doc-owned memory
```

Copy the string first when it must outlive the document:

```c
const char *name_in_doc = yyjson_get_str(name_val);
char *name_copy = strdup(name_in_doc);

yyjson_doc_free(doc);

printf("%s\n", name_copy);
free(name_copy);
```

The official documentation states that immutable `yyjson_val` objects have the same lifetime as their owning document and cannot be freed individually. ([Ibireme](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html "yyjson: yyjson.h File Reference"))

## `yyjson_doc` versus `yyjson_val`

`yyjson_doc` is the memory-owning container:

```c
yyjson_doc *doc;
```

`yyjson_val` represents one JSON value:

```c
yyjson_val *value;
```

For this JSON:

```json
{
  "name": "Alice",
  "scores": [90, 85]
}
```

The document contains multiple values:

```text
document
└── object
    ├── string: "Alice"
    └── array
        ├── number: 90
        └── number: 85
```

The root object, array, strings, and numbers are all `yyjson_val` objects owned by the same `yyjson_doc`.

## Immutable versus mutable documents

`yyjson_doc` is intended primarily for reading parsed JSON:

```c
yyjson_doc *doc;
yyjson_val *val;
```

For constructing or modifying JSON, yyjson uses:

```c
yyjson_mut_doc *doc;
yyjson_mut_val *val;
```

So the practical distinction is:

```text
yyjson_doc       parsed, immutable JSON
yyjson_mut_doc   buildable or modifiable JSON
```

In summary, `yyjson_doc` is the **owner and lifetime container for the complete parsed JSON tree**. You parse into it, retrieve the root, inspect values through `yyjson_val *`, and finally release everything with one call:

```c
yyjson_doc_free(doc);
```