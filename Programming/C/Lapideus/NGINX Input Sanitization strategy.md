
Great question. Here's a breakdown to guide your design decision:

---

## 🔹 Error Handling: What Are You Referring To?

You likely mean **runtime error logging** (e.g. logging failed memory allocation, invalid state transitions, system call errors), _not_ input validation or strict sanitization (like checking if a header is malformed).

These serve **different purposes** and thus belong in **different layers/files**.

---

## ✅ Recommended Separation of Concerns

### 1. **Low-Level Modules (e.g. `http_request.c`)**

- **Responsibility**: Implement core logic, input parsing, protocol handling, and internal state management.
    
- **What goes here:**
    
    - Validate inputs **specific to the module’s logic** (e.g., malformed request line, null pointers passed to internal routines).
        
    - Return error codes or NULL on failure.
        
    - **Avoid logging here unless critical**. This should usually bubble up to a higher level.
        
- **Avoid:** Calling `log_error()` directly unless the module owns the context (like connections or requests).
    

---

### 2. **Mid-Level/Control Code (e.g. HTTP handler, state machine)**

- **Responsibility**: Control flow, error routing, and coordination.
    
- **What goes here:**
    
    - Interpret return codes from low-level functions.
        
    - Handle fallbacks, cleanups, retries.
        
    - **Log errors here**, because this is where you have access to:
        
        - The `ngx_log_t` or your equivalent log context.
            
        - The connection/request associated with the error.
            
        - The error's impact (can you recover, or should you close the connection?).
            

---

### 3. **Logging Layer (e.g. `log.c`)**

- **Responsibility**: Formatting and writing error/debug/info logs.
    
- **What goes here:**
    
    - `log_error_core()`, `log_debug()`, etc.
        
    - Severity formatting (`[error]`, `[warn]`, `[info]`)
        
    - Writing to files or stderr.
        
- **No sanitization or branching logic** should be here — it’s just for formatting and dispatch.
    

---

## 🧠 Analogy with NGINX

NGINX rarely logs directly in modules like `ngx_http_request.c`. Instead:

- It returns `NGX_ERROR`, `NGX_DECLINED`, etc.
    
- The _caller_ (e.g. phase handlers or the main event loop) logs using `ngx_log_error()` with the appropriate `ngx_log_t`.
    

Example:

```c
if (parse_result != NGX_OK) {
    ngx_log_error(NGX_LOG_ERR, r->connection->log, 0,
                  "invalid request line");
    return NGX_HTTP_BAD_REQUEST;
}
```

---

## ✅ Bottom Line

|Layer|Error Checking|Logging|
|---|---|---|
|**Low-level (e.g. http_request.c)**|Input validation, return codes|No (except fatal internal logic bugs)|
|**Mid-level (e.g. FSM, request handling)**|Interpret errors, route logic|Yes, with full context|
|**Logging Layer (`log.c`)**|No|Yes – formatting and dispatch only|

Let me know if you want a pattern to implement error propagation across layers.