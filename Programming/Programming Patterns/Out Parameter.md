Yes, both of those changes are possible, and they map directly onto a common C technique.

Let’s break it down.

---

## 1. Your current function

Right now you have:

```

mongo_status_t mongo_connect(const char* uri) {    mongo_status_t status = (mongo_status_t){0};    /* ... fill status ... */    return status;}
```

This:

- Allocates `status` on the **stack**.
- Returns it **by value** to the caller.

In `main` you do something like:

```

mongo_status_t status = mongo_connect(uri);
```

So the caller gets a _copy_ of that struct.

---

## 2. Changing the return type to `int` and using `mongo_status_t *` (or `mongo_status_t **`)

There are two common patterns here.

### Pattern A: Return `int` and fill a struct via pointer (most idiomatic)

Change the function signature to:

```

int mongo_connect(const char* uri, mongo_status_t* out_status);
```

Usage:

```

mongo_status_t status;int rc = mongo_connect(uri, &status);if (rc != 0) {    // handle error}
```

Inside the function:

```

int mongo_connect(const char* uri, mongo_status_t* out_status) {    // zero-initialize the struct    *out_status = (mongo_status_t){0};    if (uri == NULL || uri[0] == '\0') {        out_status->error_code = 1;        /* ... */        return 1; // error code    }    /* ... on success ... */    out_status->connected = 1;    /* ... */    return 0; // success}
```

**What technique is this?**

- Using an **“out parameter”** (output parameter).
- It’s also often described as a way to simulate **pass‑by‑reference** in C.
- Very common in C APIs: function returns an error code, arguments are filled with data.

### Pattern B: Use `mongo_status_t **` and allocate inside

If you want the function itself to **allocate** the struct and give ownership to the caller, you could do:

```

int mongo_connect(const char* uri, mongo_status_t** out_status);
```

Usage:

```

mongo_status_t* status = NULL;int rc = mongo_connect(uri, &status);if (rc != 0) {    // handle error} else {    // use *status}free(status);
```

Inside:

```

int mongo_connect(const char* uri, mongo_status_t** out_status) {    *out_status = malloc(sizeof(**out_status));    if (*out_status == NULL) {        return -1; // allocation failure    }    mongo_status_t* status = *out_status;    *status = (mongo_status_t){0};    if (uri == NULL || uri[0] == '\0') {        status->error_code = 1;        /* ... */        return 1;    }    /* ... success ... */    status->connected = 1;    return 0;}
```

**What technique is this?**

- Still **out-parameter**, but via **pointer-to-pointer**.
- This is the classic C pattern when a function:
    - allocates memory,
    - returns ownership to the caller,
    - and separately returns an error code.

---

## 3. Which is better here?

For your `mongo_status_t`, which is a **small, fixed-size struct**, returning it **by value**:

```

mongo_status_t mongo_connect(const char* uri);
```

is already perfectly fine and clean.

If you want to **also** return a specific error code separate from the struct, the most idiomatic and professional pattern is **Pattern A**:

```

int mongo_connect(const char* uri, mongo_status_t* out_status);
```

Pattern B (`mongo_status_t**` and dynamic allocation) is more appropriate when:

- The size of the object is large or variable,
- Or the function needs to control the lifetime / creation of the object,
- Or the object is shared across many functions as a heap-allocated handle.

---

## 4. About the clangd errors

The errors:

- `Unknown type name 'i'`
- `Expected ';'` at the function line

usually mean there’s **some syntax problem before** the function (often a stray character like `i`, or a missing `;` or `}`) that causes the parser to get confused and mis-read `mongo_status_t` as something else.

Those are unrelated to changing the return type; they’re typically fixed by checking the lines immediately **before** the function for:

- A missing `;` after a struct/typedef,
- A stray character (like `i` alone on a line),
- An unclosed brace `}`.

---

## 5. To answer your conceptual question directly

> would that be possible and what technique that this makes me think of would this be

- **Yes, it is possible**:
    
    - change the return type to `int`,
    - take either `mongo_status_t*` or `mongo_status_t**` as an argument,
    - and modify the caller-visible value.
- The technique is:
    
    - **Using out-parameters via pointers**, or
    - Conceptually, **simulating pass‑by‑reference** in C,
    - With `mongo_status_t**` specifically, you’re using a **pointer-to-pointer out-parameter** to allow the function to allocate and assign a new pointer for the caller.

If you tell me which style you want to try (`int` + `mongo_status_t*` vs `int` + `mongo_status_t**`), I can write the exact function + call site versions for your case.