Related to [[C]]
#malloc_function #C 

**Function Signature**

`void *malloc(size_t size);`

Parameters:

- `size` — the number of bytes to allocate.

Return value:

- Returns a pointer to the allocated memory (`void *`) on success.
- Returns `NULL` if the allocation fails.

## Personal Notes 

- When allocating a struct through a pointer, consider `struct example_t *example`. You can allocate enough memory for the object that `example` points to with:
	`example = malloc(sizeof(*example));`
	
	`sizeof *example` means “the size of the struct object pointed to by `example`.”

