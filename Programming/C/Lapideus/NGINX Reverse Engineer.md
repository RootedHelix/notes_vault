

# NGINX HTTP Struct
Excellent question — and you're now thinking exactly like the designers of NGINX.

Let’s walk through how **NGINX stores the parsed pieces of an HTTP request** in **lightweight structs (`ngx_str_t`, `ngx_table_elt_t`)**, and how these are then **composed into a master struct** like `ngx_http_request_t`.

---

## ✅ The Design Pattern

NGINX follows this principle:

> **Parse pieces → wrap each in lightweight structs → store pointers to those structs inside a centralized request object.**

This allows:

- Clear separation of concerns
    
- Easy reuse and extension
    
- Consistent, fast access to fields
    

---

## 🧱 Step-by-Step Example

### 1. **Header Representation (`ngx_table_elt_t`)**

Each header is stored like this:

```c
typedef struct {
    ngx_uint_t  hash;
    ngx_str_t   key;    // e.g., "Host"
    ngx_str_t   value;  // e.g., "example.com"
    u_char     *lowcase_key;
} ngx_table_elt_t;
```

Each header field is a small struct — easy to link or store.

---

### 2. **Grouped Headers (`ngx_http_headers_in_t`)**

NGINX groups **known headers** in a dedicated struct like:

```c
typedef struct {
    ngx_table_elt_t  *host;
    ngx_table_elt_t  *user_agent;
    ngx_table_elt_t  *content_length;
    ngx_list_t        headers;  // list of all headers (key-value)
} ngx_http_headers_in_t;
```

This gives:

- Fast direct access to common headers (e.g., `r->headers_in.host->value`)
    
- A general list for others (e.g., `X-Custom-Whatever`)
    

---

### 3. **The Master Request Object (`ngx_http_request_t`)**

Now all the pieces are composed into the main request struct:

```c
typedef struct ngx_http_request_s {
    ngx_str_t                 method_name;  // e.g., "GET"
    ngx_str_t                 uri;          // e.g., "/index.html"
    ngx_str_t                 args;         // e.g., "id=123"

    ngx_http_headers_in_t     headers_in;   // request headers
    ngx_http_headers_out_t    headers_out;  // response headers

    ngx_http_request_body_t  *request_body;

    ngx_pool_t               *pool;         // memory pool for this request
    ngx_connection_t         *connection;   // client connection
    ...
} ngx_http_request_t;
```

This struct is created **per request**, and it stores:

- Lightweight, length-tracked fields
    
- Pointers to parsed data
    
- Everything needed to process the request in one place
    

---

## 🔧 Parsing Flow Summary

1. Receive raw buffer from socket
    
2. Tokenize/parse components (method, URI, headers)
    
3. Allocate memory from pool and wrap each in `ngx_str_t` or `ngx_table_elt_t`
    
4. Assign fields into `ngx_http_request_t`
    

---

## ✅ TL;DR:

> NGINX stores parsed pieces (e.g., method, URI, headers) in lightweight structs like `ngx_str_t` or `ngx_table_elt_t`, and then **composes those into a master `ngx_http_request_t` struct**.  
> This keeps memory use efficient, access fast, and logic modular — all backed by a dedicated memory pool.

Would you like a simplified C version of this kind of `http_request_t` struct that you could use in your own server?


## Commentary

- so when you assign the fields in the http_request struct you are simply assigning the already allocated memory to the fields

## NGINX log.c linux only

### FULL 
 * `Copyright (C) Igor Sysoev
 * `
 * `Copyright (C) Nginx, Inc.`
 

`#include <ngx_config.h>`
`#include <ngx_core.h>`

`static char *ngx_error_log(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);`
`static char *ngx_log_set_levels(ngx_conf_t *cf, ngx_log_t *log);`
`static void ngx_log_insert(ngx_log_t *log, ngx_log_t *new_log);`

`static ngx_command_t  ngx_errlog_commands[] = {`
    `{ ngx_string("error_log"),`
      `NGX_MAIN_CONF|NGX_CONF_1MORE,`
      `ngx_error_log,`
      `0,`
      `0,`
      `NULL },`

    `ngx_null_command`
`};`

`static ngx_core_module_t  ngx_errlog_module_ctx = {`
    `ngx_string("errlog"),`
    `NULL,`
    `NULL`
`};`

`ngx_module_t  ngx_errlog_module = {`
    `NGX_MODULE_V1,`
    `&ngx_errlog_module_ctx,`
    `ngx_errlog_commands,`
    `NGX_CORE_MODULE,`
    `NULL,`
    `NULL,`
    `NULL,`
    `NULL,`
    `NULL,`
    `NULL,`
    `NULL,`
    `NGX_MODULE_V1_PADDING`
`};`

`static ngx_log_t        ngx_log;`
`static ngx_open_file_t  ngx_log_file;`
`ngx_uint_t              ngx_use_stderr = 1;`

`static ngx_str_t err_levels[] = {`
    `ngx_null_string,`
    `ngx_string("emerg"),`
    `ngx_string("alert"),`
    `ngx_string("crit"),`
    `ngx_string("error"),`
    `ngx_string("warn"),`
    `ngx_string("notice"),`
    `ngx_string("info"),`
    `ngx_string("debug")`
`};`

`static const char *debug_levels[] = {`
    `"debug_core", "debug_alloc", "debug_mutex", "debug_event",`
    `"debug_http", "debug_mail", "debug_stream"`
`};`

`void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err,`
    `const char *fmt, ...)`
`{`
    `va_list      args;`
    `u_char      *p, *last, *msg;`
    `ssize_t      n;`
    `ngx_uint_t   wrote_stderr, debug_connection;`
    `u_char       errstr[NGX_MAX_ERROR_STR];`

    t`last = errstr + NGX_MAX_ERROR_STR;`

    `p = ngx_cpymem(errstr, ngx_cached_err_log_time.data,`
                   `ngx_cached_err_log_time.len);`

    `p = ngx_slprintf(p, last, " [%V] ", &err_levels[level]);`
    `p = ngx_slprintf(p, last, "%P#%T: ", ngx_log_pid, ngx_log_tid);`

    `if (log->connection) {`
        `p = ngx_slprintf(p, last, "*%uA ", log->connection);`
    `}`

    `msg = p;`

    `va_start(args, fmt);`
    `p = ngx_vslprintf(p, last, fmt, args);`
    `va_end(args);`

    `if (err) {`
        `p = ngx_log_errno(p, last, err);`
    `}`

    `if (level != NGX_LOG_DEBUG && log->handler) {`
        `p = log->handler(log, p, last - p);`
    `}`

    `if (p > last - NGX_LINEFEED_SIZE) {`
        `p = last - NGX_LINEFEED_SIZE;`
    `}`

    `ngx_linefeed(p);`

    `wrote_stderr = 0;`
    `debug_connection = (log->log_level & NGX_LOG_DEBUG_CONNECTION) != 0;`

    `while (log) {`
        `if (log->log_level < level && !debug_connection) {`
            `break;`
        `}`

        `if (log->writer) {`
            `log->writer(log, level, errstr, p - errstr);`
            `goto next;`
        `}`

        `if (ngx_time() == log->disk_full_time) {`
            `goto next;`
        `}`

        `n = ngx_write_fd(log->file->fd, errstr, p - errstr);`

        `if (n == -1 && ngx_errno == NGX_ENOSPC) {`
            `log->disk_full_time = ngx_time();`
        `}`

        `if (log->file->fd == ngx_stderr) {`
            `wrote_stderr = 1;`
        `}`

    `next:`
        `log = log->next;`
    `}`

    `if (!ngx_use_stderr || level > NGX_LOG_WARN || wrote_stderr) {`
        `return;`
    `}`

    `msg -= (7 + err_levels[level].len + 3);`
    `(void) ngx_sprintf(msg, "nginx: [%V] ", &err_levels[level]);`
    `(void) ngx_write_console(ngx_stderr, msg, p - msg);`
`}`

`void ngx_cdecl ngx_log_abort(ngx_err_t err, const char *fmt, ...)`
`{`
    `u_char   *p;`
    `va_list   args;`
    `u_char    errstr[NGX_MAX_CONF_ERRSTR];`

    `va_start(args, fmt);`
    `p = ngx_vsnprintf(errstr, sizeof(errstr) - 1, fmt, args);`
    `va_end(args);`

    `ngx_log_error(NGX_LOG_ALERT, ngx_cycle->log, err, "%*s", p - errstr, errstr);`
`}`

`void ngx_cdecl ngx_log_stderr(ngx_err_t err, const char *fmt, ...)`
`{`
    `u_char   *p, *last;`
    `va_list   args;`
    `u_char    errstr[NGX_MAX_ERROR_STR];`

    `last = errstr + NGX_MAX_ERROR_STR;`

    `p = ngx_cpymem(errstr, "nginx: ", 7);`
    `va_start(args, fmt);`
    `p = ngx_vslprintf(p, last, fmt, args);`
    `va_end(args);`

    `if (err) {`
        `p = ngx_log_errno(p, last, err);`
    `}`

    `if (p > last - NGX_LINEFEED_SIZE) {`
        `p = last - NGX_LINEFEED_SIZE;`
    `}`

    `ngx_linefeed(p);`
    `(void) ngx_write_console(ngx_stderr, errstr, p - errstr);`
`}`

`u_char *ngx_log_errno(u_char *buf, u_char *last, ngx_err_t err)`
`{`
    `if (buf > last - 50) {`
        `buf = last - 50;`
        `*buf++ = '.'; *buf++ = '.'; *buf++ = '.';`
    `}`

    `buf = ngx_slprintf(buf, last, " (%d: ", err);`
    `buf = ngx_strerror(err, buf, last - buf);`

    `if (buf < last) {`
        `*buf++ = ')';`
    `}`

    `return buf;`
`}`

`ngx_log_t *ngx_log_init(u_char *prefix, u_char *error_log)`
`{`
    `u_char  *p, *name;`
    `size_t   nlen, plen;`

    `ngx_log.file = &ngx_log_file;`
    `ngx_log.log_level = NGX_LOG_NOTICE;`

    `if (error_log == NULL) {`
        `error_log = (u_char *) NGX_ERROR_LOG_PATH;`
    `}`

    `name = error_log;`
    `nlen = ngx_strlen(name);`

    `if (nlen == 0) {`
        `ngx_log_file.fd = ngx_stderr;`
        `return &ngx_log;`
    `}`

    `p = NULL;`

    `if (name[0] != '/') {`
        `if (prefix) {`
            `plen = ngx_strlen(prefix);`
        `} else {`
`#ifdef NGX_PREFIX`
            `prefix = (u_char *) NGX_PREFIX;`
            `plen = ngx_strlen(prefix);`
`#else`
            `plen = 0;`
`#endif`
        `}`

        `if (plen) {`
            `name = malloc(plen + nlen + 2);`
            `if (name == NULL) return NULL;`
            `p = ngx_cpymem(name, prefix, plen);`
            `if (!ngx_path_separator(*(p - 1))) *p++ = '/';`
            `ngx_cpystrn(p, error_log, nlen + 1);`
            `p = name;`
        `}`
    `}`

    `ngx_log_file.fd = ngx_open_file(name, NGX_FILE_APPEND,`
                                    `NGX_FILE_CREATE_OR_OPEN,`
                                    `NGX_FILE_DEFAULT_ACCESS);`

    `if (ngx_log_file.fd == NGX_INVALID_FILE) {`
        `ngx_log_stderr(ngx_errno, "[alert] could not open error log file: " ngx_open_file_n " \"%s\" failed", name);`
        `ngx_log_file.fd = ngx_stderr;`
    `}`

    `if (p) ngx_free(p);`
    `return &ngx_log;`
`}`

`ngx_int_t ngx_log_open_default(ngx_cycle_t *cycle)`
`{`
    `ngx_log_t  *log;`

    `if (ngx_log_get_file_log(&cycle->new_log) != NULL) return NGX_OK;`

    `if (cycle->new_log.log_level != 0) {`
        `log = ngx_pcalloc(cycle->pool, sizeof(ngx_log_t));`
        `if (log == NULL) return NGX_ERROR;`
    `} else {`
        `log = &cycle->new_log;`
    `}`

    `log->log_level = NGX_LOG_ERR;`
    `log->file = ngx_conf_open_file(cycle, &cycle->error_log);`
    `if (log->file == NULL) return NGX_ERROR;`
    `if (log != &cycle->new_log) ngx_log_insert(&cycle->new_log, log);`
    `return NGX_OK;`
`}`

`ngx_int_t ngx_log_redirect_stderr(ngx_cycle_t *cycle)`
`{`
    `ngx_fd_t  fd;`

    `if (cycle->log_use_stderr) return NGX_OK;`

    `fd = ngx_log_get_file_log(cycle->log)->file->fd;`
    `if (fd != ngx_stderr) {`
        `if (ngx_set_stderr(fd) == NGX_FILE_ERROR) {`
            `ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno, ngx_set_stderr_n " failed");`
            `return NGX_ERROR;`
        `}`
    `}`

    `return NGX_OK;`
`}`

`ngx_log_t *ngx_log_get_file_log(ngx_log_t *head)`
`{`
    `for (; head; head = head->next) {`
        `if (head->file != NULL) return head;`
    `}`
    `return NULL;`
`}`

`static char *ngx_log_set_levels(ngx_conf_t *cf, ngx_log_t *log)`
`{`
    `ngx_uint_t i, n, d, found;`
    `ngx_str_t *value = cf->args->elts;`

    `if (cf->args->nelts == 2) {`
        `log->log_level = NGX_LOG_ERR;`
        `return NGX_CONF_OK;`
    `}`

    `for (i = 2; i < cf->args->nelts; i++) {`
        `found = 0;`
        `for (n = 1; n <= NGX_LOG_DEBUG; n++) {`
            `if (ngx_strcmp(value[i].data, err_levels[n].data) == 0) {`
                `if (log->log_level != 0) {`
                    `ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "duplicate log level \"%V\"", &value[i]);`
                    `return NGX_CONF_ERROR;`
                `}`
                `log->log_level = n;`
                `found = 1;`
                `break;`
            `}`
        `}`
        `for (n = 0, d = NGX_LOG_DEBUG_FIRST; d <= NGX_LOG_DEBUG_LAST; d <<= 1) {`
            `if (ngx_strcmp(value[i].data, debug_levels[n++]) == 0) {`
                `if (log->log_level & ~NGX_LOG_DEBUG_ALL) {`
                    `ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "invalid log level \"%V\"", &value[i]);`
                    `return NGX_CONF_ERROR;`
                `}`
                `log->log_level |= d;`
                `found = 1;`
                `break;`
            `}`
        `}`
        `if (!found) {`
            `ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "invalid log level \"%V\"", &value[i]);`
            `return NGX_CONF_ERROR;`
        `}`
    `}`

    `if (log->log_level == NGX_LOG_DEBUG) {`
        `log->log_level = NGX_LOG_DEBUG_ALL;`
    `}`

    `return NGX_CONF_OK;`
`}`

`static char *ngx_error_log(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)`
`{`
    `ngx_log_t *dummy = &cf->cycle->new_log;`
    `return ngx_log_set_log(cf, &dummy);`
`}`  

`char *ngx_log_set_log(ngx_conf_t *cf, ngx_log_t **head); // Refer to full file if you need all cases (syslog, memory etc.)`


### No Modules

/*
 * ==Minimal Linux-Only ngx_log.c (without module system)==
 * ==Based on NGINX source code, preserves original function logic and naming==
 * ==License: BSD-like (retain copyright notices)==
 */

#include <ngx_config.h>
#include <ngx_core.h>

static ngx_log_t        ngx_log;
static ngx_open_file_t  ngx_log_file;
ngx_uint_t              ngx_use_stderr = 1;

static ngx_str_t err_levels[] = {
    ngx_null_string,
    ngx_string("emerg"),
    ngx_string("alert"),
    ngx_string("crit"),
    ngx_string("error"),
    ngx_string("warn"),
    ngx_string("notice"),
    ngx_string("info"),
    ngx_string("debug")
};

void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err,
    const char *fmt, ...)
{
    va_list      args;
    u_char      *p, *last, *msg;
    ssize_t      n;
    ngx_uint_t   wrote_stderr, debug_connection;
    u_char       errstr[NGX_MAX_ERROR_STR];

    last = errstr + NGX_MAX_ERROR_STR;

    p = ngx_cpymem(errstr, ngx_cached_err_log_time.data,
                   ngx_cached_err_log_time.len);

    p = ngx_slprintf(p, last, " [%V] ", &err_levels[level]);
    p = ngx_slprintf(p, last, "%P#%T: ", ngx_log_pid, ngx_log_tid);

    if (log->connection) {
        p = ngx_slprintf(p, last, "*%uA ", log->connection);
    }

    msg = p;

    va_start(args, fmt);
    p = ngx_vslprintf(p, last, fmt, args);
    va_end(args);

    if (err) {
        p = ngx_log_errno(p, last, err);
    }

    if (level != NGX_LOG_DEBUG && log->handler) {
        p = log->handler(log, p, last - p);
    }

    if (p > last - NGX_LINEFEED_SIZE) {
        p = last - NGX_LINEFEED_SIZE;
    }

    ngx_linefeed(p);

    wrote_stderr = 0;
    debug_connection = (log->log_level & NGX_LOG_DEBUG_CONNECTION) != 0;

    while (log) {
        if (log->log_level < level && !debug_connection) {
            break;
        }

        if (log->writer) {
            log->writer(log, level, errstr, p - errstr);
            goto next;
        }

        if (ngx_time() == log->disk_full_time) {
            goto next;
        }

        n = ngx_write_fd(log->file->fd, errstr, p - errstr);

        if (n == -1 && ngx_errno == NGX_ENOSPC) {
            log->disk_full_time = ngx_time();
        }

        if (log->file->fd == ngx_stderr) {
            wrote_stderr = 1;
        }

    next:
        log = log->next;
    }

    if (!ngx_use_stderr || level > NGX_LOG_WARN || wrote_stderr) {
        return;
    }

    msg -= (7 + err_levels[level].len + 3);
    (void) ngx_sprintf(msg, "nginx: [%V] ", &err_levels[level]);
    (void) ngx_write_console(ngx_stderr, msg, p - msg);
}

void ngx_cdecl ngx_log_abort(ngx_err_t err, const char *fmt, ...)
{
    u_char   *p;
    va_list   args;
    u_char    errstr[NGX_MAX_CONF_ERRSTR];

    va_start(args, fmt);
    p = ngx_vsnprintf(errstr, sizeof(errstr) - 1, fmt, args);
    va_end(args);

    ngx_log_error(NGX_LOG_ALERT, ngx_cycle->log, err, "%*s", p - errstr, errstr);
}

void ngx_cdecl ngx_log_stderr(ngx_err_t err, const char *fmt, ...)
{
    u_char   *p, *last;
    va_list   args;
    u_char    errstr[NGX_MAX_ERROR_STR];

    last = errstr + NGX_MAX_ERROR_STR;

    p = ngx_cpymem(errstr, "nginx: ", 7);
    va_start(args, fmt);
    p = ngx_vslprintf(p, last, fmt, args);
    va_end(args);

    if (err) {
        p = ngx_log_errno(p, last, err);
    }

    if (p > last - NGX_LINEFEED_SIZE) {
        p = last - NGX_LINEFEED_SIZE;
    }

    ngx_linefeed(p);
    (void) ngx_write_console(ngx_stderr, errstr, p - errstr);
}

u_char *ngx_log_errno(u_char *buf, u_char *last, ngx_err_t err)
{
    if (buf > last - 50) {
        buf = last - 50;
        *buf++ = '.'; *buf++ = '.'; *buf++ = '.';
    }

    buf = ngx_slprintf(buf, last, " (%d: ", err);
    buf = ngx_strerror(err, buf, last - buf);

    if (buf < last) {
        *buf++ = ')';
    }

    return buf;
}

ngx_log_t *ngx_log_init(u_char *prefix, u_char *error_log)
{
    u_char  *p, *name;
    size_t   nlen, plen;

    ngx_log.file = &ngx_log_file;
    ngx_log.log_level = NGX_LOG_NOTICE;

    if (error_log == NULL) {
        error_log = (u_char *) NGX_ERROR_LOG_PATH;
    }

    name = error_log;
    nlen = ngx_strlen(name);

    if (nlen == 0) {
        ngx_log_file.fd = ngx_stderr;
        return &ngx_log;
    }

    p = NULL;

    if (name[0] != '/') {
        if (prefix) {
            plen = ngx_strlen(prefix);
        } else {
#ifdef NGX_PREFIX
            prefix = (u_char *) NGX_PREFIX;
            plen = ngx_strlen(prefix);
#else
            plen = 0;
#endif
        }

        if (plen) {
            name = malloc(plen + nlen + 2);
            if (name == NULL) return NULL;
            p = ngx_cpymem(name, prefix, plen);
            if (!ngx_path_separator(*(p - 1))) *p++ = '/';
            ngx_cpystrn(p, error_log, nlen + 1);
            p = name;
        }
    }

    ngx_log_file.fd = ngx_open_file(name, NGX_FILE_APPEND,
                                    NGX_FILE_CREATE_OR_OPEN,
                                    NGX_FILE_DEFAULT_ACCESS);

    if (ngx_log_file.fd == NGX_INVALID_FILE) {
        ngx_log_stderr(ngx_errno, "[alert] could not open error log file: " ngx_open_file_n " \"%s\" failed", name);
        ngx_log_file.fd = ngx_stderr;
    }

    if (p) ngx_free(p);
    return &ngx_log;
}

ngx_int_t ngx_log_redirect_stderr(ngx_cycle_t *cycle)
{
    ngx_fd_t  fd;

    if (cycle->log_use_stderr) return NGX_OK;

    fd = ngx_log_get_file_log(cycle->log)->file->fd;
    if (fd != ngx_stderr) {
        if (ngx_set_stderr(fd) == NGX_FILE_ERROR) {
            ngx_log_error(NGX_LOG_ALERT, cycle->log, ngx_errno, ngx_set_stderr_n " failed");
            return NGX_ERROR;
        }
    }

    return NGX_OK;
}

ngx_log_t *ngx_log_get_file_log(ngx_log_t *head)
{
    for (; head; head = head->next) {
        if (head->file != NULL) return head;
    }
    return NULL;
}

/*
==Minimal Design no modules, nginx compleity extracted and for baseline logging==
==*/==
==// minimal_log.c==
==// Trimmed-down version of your paste: single sink (file or stderr), levels, errno text.==

#include <stdio.h>
#include <stdarg.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/syscall.h>

// --- Minimal compat typedefs (adjust to your project types if you already have them)
typedef int            ngx_int_t;
typedef unsigned int   ngx_uint_t;
typedef int            ngx_err_t;

typedef struct {
    int fd; // -1 means stderr
} ngx_open_file_t;

typedef struct ngx_log_s {
    ngx_open_file_t *file;  // required
    ngx_uint_t       log_level; // minimum level to write
    int              log_use_stderr; // if file missing, write to stderr
} ngx_log_t;

// --- Globals kept (simple)
static ngx_log_t        ngx_log;
static ngx_open_file_t  ngx_log_file;
ngx_uint_t              ngx_use_stderr = 1;

// --- Levels (keep)
static const char *err_levels[] = {
    "", "emerg","alert","crit","error","warn","notice","info","debug"
};

// --- Helpers
static inline pid_t get_tid(void) {
#ifdef SYS_gettid
    return (pid_t)syscall(SYS_gettid);
#else
    return (pid_t)getpid();
#endif
}

// Appends " (errno: text)" if space
static unsigned char *ngx_log_errno(unsigned char *buf, unsigned char *last, ngx_err_t err) {
    if (err == 0) return buf;

    if (buf > last - 50) { // reserve a little space
        buf = last - 50;
        *buf++='.'; *buf++='.'; *buf++='.';
    }

    int n = snprintf((char*)buf, (size_t)(last - buf), " (%d: %s)", err, strerror(err));
    if (n < 0) return buf;
    buf += (n < (last - buf)) ? n : (last - buf);
    return buf;
}

// --- Core logging: one buffer, one write
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log, ngx_err_t err, const char *fmt, ...)
{
    if (!log) return;
    if (level > 8) level = 8; // clamp
    if (level == 0) level = 1;

    if (level > log->log_level) return;

    // Fixed buffer
    enum { MAX_ERR = 4096 };
    unsigned char errstr[MAX_ERR];
    unsigned char *p = errstr, *last = errstr + MAX_ERR;

    // timestamp
    struct timespec ts; clock_gettime(CLOCK_REALTIME, &ts);
    struct tm tm; localtime_r(&ts.tv_sec, &tm);
    char tbuf[32];
    size_t tlen = strftime(tbuf, sizeof tbuf, "%Y/%m/%d %H:%M:%S", &tm);
    if (tlen > 0) {
        int n = snprintf((char*)p, (size_t)(last - p), "%.*s", (int)tlen, tbuf);
        if (n > 0) p += (n < (last - p)) ? n : (last - p);
    }

    // level + pid/tid
    int n = snprintf((char*)p, (size_t)(last - p), " [%s] %d#%d: ",
                     err_levels[level], (int)getpid(), (int)get_tid());
    if (n > 0) p += (n < (last - p)) ? n : (last - p);

    // message
    va_list args;
    va_start(args, fmt);
    n = vsnprintf((char*)p, (size_t)(last - p), fmt, args);
    va_end(args);
    if (n > 0) p += (n < (last - p)) ? n : (last - p);

    // errno text
    p = ngx_log_errno(p, last, err);

    // newline (ensure space)
    if (p > last - 2) p = last - 2;
    *p++ = '\n';

    // choose fd
    int fd = (log->file && log->file->fd >= 0) ? log->file->fd : -1;
    if (fd < 0 && (log->log_use_stderr || ngx_use_stderr)) {
        fd = STDERR_FILENO;
    }
    if (fd >= 0) {
        (void)write(fd, errstr, (size_t)(p - errstr)); // best-effort
    }
}

// --- Init a single file sink (or fallback to stderr)
ngx_log_t *ngx_log_init(const char *error_log_path, ngx_uint_t min_level)
{
    ngx_log.file = &ngx_log_file;
    ngx_log.log_level = min_level;
    ngx_log.log_use_stderr = 1;

    if (!error_log_path || error_log_path[0] == '\0') {
        ngx_log_file.fd = -1; // use stderr
        return &ngx_log;
    }

    int fd = open(error_log_path, O_WRONLY | O_CREAT | O_APPEND, 0644);
    if (fd == -1) {
        // fallback to stderr; also print one alert so user notices
        ngx_log_file.fd = -1;
        ngx_log_error_core(2 /*alert*/, &ngx_log, errno,
                           "could not open error log file \"%s\"", error_log_path);
        return &ngx_log;
    }

    ngx_log_file.fd = fd;
    return &ngx_log;
}

// --- Optional: close on shutdown (call at exit)
void ngx_log_close(ngx_log_t *log) {
    if (log && log->file && log->file->fd >= 0) {
        close(log->file->fd);
        log->file->fd = -1;
    }
}

