Related to [[C]], [[Data Structures]],
#arrays #data_structures 

### General 

There are some multiple labels that are placed on arrays such as **array**, **table**, **registry**, or **dispatch table**

#### Table

Use **table** when you want to emphasize:

- it’s being used for lookup
- each slot corresponds to something meaningful
- it’s part of a dispatch mechanism


### Fixed Size Arrays 

- Size known at runtime
syntax:
`unsigned int size = 20;
int array[size];`

### VLAs (Variable Length Arrays)

- Syntax: `size_t n = get_n(); int a[n];` // `n` known at runtime

- Risky for large `n` (stack overflow).
    
- Not allowed with static storage duration and not allowed as struct members.

### Dynamic Array
- Use Case: when you need to iterate/sort/search more than you need to insert/remove, mutate
- Heap Allocated with `malloc` | `realloc` | `calloc` | `free`
Syntax:
`size_t n = get_n();`
`int *a = malloc(n * sizeof *a);`
`// use a[0..n-1]`
`free(a);`
