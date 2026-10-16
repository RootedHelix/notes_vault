Related to [[yysjon]], [[JSON]], [[C]]
#yyjson #json #yyjson_is_object

## Personal Notes

`yyjson_is_object()` checks whether a `yyjson_val` represents a JSON object.

**Function Signature**

```c
bool yyjson_is_object(const yyjson_val *val);
```

It returns:

- `true` if `val` is a JSON object
    
- `false` if `val` is another JSON type or `NULL`

## AI Response 


Example:

```c
yyjson_doc *doc = yyjson_read_file("data.json", 0, NULL, NULL);
if (doc == NULL) {
    return 1;
}

yyjson_val *root = yyjson_doc_get_root(doc);

if (yyjson_is_object(root)) {
    printf("The root value is a JSON object.\n");
} else {
    printf("The root value is not a JSON object.\n");
}

yyjson_doc_free(doc);
```

For this JSON:

```json
{
  "name": "Harrison",
  "age": 25
}
```

`yyjson_is_object(root)` returns `true`.

For this JSON:

```json
[
  1,
  2,
  3
]
```

it returns `false`, because the root value is an array.

You commonly use it before calling object-specific functions:

```c
if (yyjson_is_object(root)) {
    yyjson_val *name = yyjson_obj_get(root, "name");
}
```

This prevents treating an array, string, number, or other JSON value as an object.

