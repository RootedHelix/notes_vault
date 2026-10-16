Related to [[yyjson]], [[JSON]], [[C]]
#yyjson #yyjson_val #C 

## Personal Notes


`yyjson_val` represents **one immutable JSON value** inside a parsed `yyjson_doc`.

**Struct Definition**

```C
typedef union yyjson_val_uni {    
	uint64_t    u64;    
	int64_t     i64;    
	double      f64;    
	const char *str;    
	void       *ptr;    
	size_t      ofs;
	} yyjson_val_uni;

struct yyjson_val {    
uint64_t       tag;  /* type, subtype, and length */
yyjson_val_uni uni;  /* payload */
};
```

## AI Response



A `yyjson_doc` owns the complete parsed document, while each individual object, array, string, number, boolean, or `null`is represented by a `yyjson_val`.

```c
yyjson_doc *doc = yyjson_read_file("data.json", 0, NULL, NULL);

yyjson_val *root = yyjson_doc_get_root(doc);
```

Here:

- `doc` is the whole parsed document.
    
- `root` is one `yyjson_val` representing the top-level JSON value.
    

## Example JSON

```json
{
  "name": "Alice",
  "age": 30,
  "active": true,
  "scores": [90, 85]
}
```

Internally, the values are conceptually:

```text
yyjson_val: object
├── yyjson_val: string "Alice"
├── yyjson_val: integer 30
├── yyjson_val: boolean true
└── yyjson_val: array
    ├── yyjson_val: integer 90
    └── yyjson_val: integer 85
```

Every JSON value is represented by the same `yyjson_val` type. Its internal tag records which JSON type it currently contains.

## Retrieving values

```c
yyjson_val *root = yyjson_doc_get_root(doc);

yyjson_val *name_val = yyjson_obj_get(root, "name");
yyjson_val *age_val = yyjson_obj_get(root, "age");
yyjson_val *active_val = yyjson_obj_get(root, "active");
```

You should check each value’s type before extracting its contents:

```c
if (yyjson_is_str(name_val)) {
    const char *name = yyjson_get_str(name_val);
    printf("Name: %s\n", name);
}

if (yyjson_is_int(age_val)) {
    int64_t age = yyjson_get_sint(age_val);
    printf("Age: %lld\n", (long long)age);
}

if (yyjson_is_bool(active_val)) {
    bool active = yyjson_get_bool(active_val);
    printf("Active: %s\n", active ? "true" : "false");
}
```

## Common type-checking functions

```c
yyjson_is_obj(val)
yyjson_is_arr(val)
yyjson_is_str(val)
yyjson_is_num(val)
yyjson_is_int(val)
yyjson_is_uint(val)
yyjson_is_real(val)
yyjson_is_bool(val)
yyjson_is_null(val)
yyjson_is_raw(val)
```

These determine what kind of JSON value the `yyjson_val` represents.

## Common getter functions

```c
yyjson_get_str(val)
yyjson_get_sint(val)
yyjson_get_uint(val)
yyjson_get_real(val)
yyjson_get_bool(val)
yyjson_get_raw(val)
```

The getter must match the value’s actual type. For example:

```c
if (yyjson_is_str(val)) {
    const char *text = yyjson_get_str(val);
}
```

Calling `yyjson_get_str()` on an integer value will not convert the integer to text.

## Objects and arrays are also `yyjson_val`

An object:

```json
{
  "name": "Alice"
}
```

is represented by a `yyjson_val` whose type is object.

An array:

```json
[1, 2, 3]
```

is represented by a `yyjson_val` whose type is array.

You can inspect them with:

```c
if (yyjson_is_obj(root)) {
    yyjson_val *name = yyjson_obj_get(root, "name");
}
```

or:

```c
if (yyjson_is_arr(root)) {
    size_t count = yyjson_arr_size(root);

    yyjson_val *first = yyjson_arr_get(root, 0);
}
```

## Lifetime and ownership

A `yyjson_val` does not independently own its memory. It belongs to the `yyjson_doc`.

```c
yyjson_val *root = yyjson_doc_get_root(doc);

yyjson_doc_free(doc);

/* root is now invalid */
```

Do not separately free a `yyjson_val`:

```c
free(root); /* Incorrect */
```

Instead, free the owning document:

```c
yyjson_doc_free(doc);
```

This invalidates every `yyjson_val *` obtained from that document.

## Immutable nature

`yyjson_val` is the immutable version of a JSON value. It is primarily used when reading parsed JSON:

```c
yyjson_doc *
yyjson_val *
```

For building or modifying JSON, yyjson uses:

```c
yyjson_mut_doc *
yyjson_mut_val *
```

So the distinction is:

```text
yyjson_val       immutable parsed value
yyjson_mut_val   mutable value used to build or modify JSON
```

## Practical validation example

```c
yyjson_read_err err;

yyjson_doc *doc = yyjson_read_file(
    "user.json",
    0,
    NULL,
    &err
);

if (doc == NULL) {
    fprintf(stderr, "Parse error: %s\n", err.msg);
    return 1;
}

yyjson_val *root = yyjson_doc_get_root(doc);

if (!yyjson_is_obj(root)) {
    fprintf(stderr, "Root must be an object\n");
    yyjson_doc_free(doc);
    return 1;
}

yyjson_val *name = yyjson_obj_get(root, "name");

if (!yyjson_is_str(name)) {
    fprintf(stderr, "\"name\" must be a string\n");
    yyjson_doc_free(doc);
    return 1;
}

printf("Name: %s\n", yyjson_get_str(name));

yyjson_doc_free(doc);
```

The central idea is:

```text
yyjson_doc = owns the complete parsed document
yyjson_val = represents one value within that document
```

A `yyjson_val` can therefore represent any JSON type, and you use the `yyjson_is_*` and `yyjson_get_*` functions to inspect it.