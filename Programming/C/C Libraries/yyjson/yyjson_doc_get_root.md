Related to [[yysjon]], [[JSON]], [[C]]
#yyjson #json 

## Personal Notes

`yyjson_doc_get_root()` returns the **top-level JSON value** stored inside a parsed `yyjson_doc`.

**Function Signature**

```C
yyjson_val *yyjson_doc_get_root(yyjson_doc *doc);
```



## AI Response


Example JSON:

```json
{
  "name": "Harrison",
  "age": 25
}
```

After parsing:

```c
yyjson_doc *doc = yyjson_read_file("data.json", 0, NULL, NULL);

if (doc == NULL) {
    // Parsing failed
    return;
}

yyjson_val *root = yyjson_doc_get_root(doc);
```

Here, `root` points to the outermost object:

```json
{
  "name": "Harrison",
  "age": 25
}
```

You can then retrieve members from it:

```c
yyjson_val *name = yyjson_obj_get(root, "name");
yyjson_val *age  = yyjson_obj_get(root, "age");

printf("Name: %s\n", yyjson_get_str(name));
printf("Age: %lld\n", (long long)yyjson_get_int(age));
```

The root is not necessarily an object. It can be any valid top-level JSON value:

```json
[1, 2, 3]
```

```json
"hello"
```

```json
42
```

```json
null
```

Therefore, you should verify its type before treating it as an object:

```c
yyjson_val *root = yyjson_doc_get_root(doc);

if (!yyjson_is_obj(root)) {
    fprintf(stderr, "Expected the root value to be an object\n");
    yyjson_doc_free(doc);
    return;
}
```

The function returns `NULL` when the provided `doc` pointer is `NULL`. The returned `yyjson_val *` is owned by the document, so you must not free it separately. It remains valid only until you call:

```c
yyjson_doc_free(doc);
```

Conceptually:

```text
yyjson_doc
    └── root yyjson_val
            ├── object members
            ├── array elements
            └── nested values
```

It is essentially the entry point for navigating the parsed JSON tree. ([ibireme.github.io](https://ibireme.github.io/yyjson/doc/doxygen/html/yyjson_8h.html?utm_source=chatgpt.com "yyjson.h File Reference"))