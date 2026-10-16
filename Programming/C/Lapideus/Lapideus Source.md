Related to [[Lapideus]], [[Lapideus Documentation]], [[Lapideus Build Log]]
#lapideus_source

The Lapideus web server is written for a Windows Pc and utilizes the GCC compiler. 
Windows PC: Using the GCC Compiler

- src files are a library not an executable

Lapideus like nginx follows an Asynchronous I/O connection model

- one Worker per Thread




# core



## `lpd_connection.h`
[[lpd_connection.h]]

## `lpd_connection.c`
[[lpd_connection.c]]
## `lpd_cycle.h`

## `lpd_palloc.h`

## `lpd_palloc.c
[[lpd_palloc.c]]

#### lpd_cycle_s

**lpd_connection_t* lpd_free_connections; 



## lpd_cycle.c

### lpd_cycle_t

- `lpd_log_t* log;` Cycle is the structure that holds the linked list of lpd_log_t structs and this is the node* head variable.


// Forward declarations
typedef struct module_s module_t;
typedef struct logger_s logger_t;
typedef struct conn_pool_s connection_pool_t;

typedef struct cycle_s {
    // Core system state (used now)
    logger_t           *logger;
    connection_pool_t  *connections;

    // Loaded modules (you can use this now)
    module_t          **modules;
    size_t              module_count;

    // Stub: reserved for module config system (used later)
    void             ***module_conf_ctx;  // Leave as NULL for now

    // Stub: optional shared context per module (also for future use)
    void              **module_ctxs;      // Can also be NULL

    // Stub: plugin system or hook registration (for future extensibility)
    void               *plugin_registry;  // Opaque type for now

} cycle_t;


#### lpd_cycle_t
- **Definition:** `src/core/ngx_cycle.h` and `ngx_cycle.c`
    
- **Purpose:** Global server context (connections, configs, logging, etc.)
    
- **References:** Included almost everywhere; especially in main, core, and event logic.

- **Details**: holds the global context of the server for the duration of the servers lifetime. 

- It refers to an entire instance of the running process context.
- A cycle persists until the server is reloaded or restarted, at which point a new cycle is created.

## lpd_log.h

## lpd_log.c

**Yes, absolutely!** Multiple `lpd_log_t` structs can point to the same `lpd_open_file_t`.

- Multiple components (http parser or other modeuls ) of the each connections have lpd_log_t* structs

- `lpd_open_file_t` = _where/how to write_: `ngx_fd_t fd`, path, optional `flush()`/`data`. Usable by logging, temp files, cache, etc.
    
- `lpd_log_t` = _when/what to write_: level threshold, connection tag, formatting/writer, etc.

### Example Scenario:

```

// One shared file
static lpd_open_file_t shared_log_file;

// Multiple log instances
static lpd_log_t error_log;
static lpd_log_t warning_log;
static lpd_log_t debug_log;

void setup_logs() {
    // All three logs write to the SAME file
    error_log.file = &shared_log_file;
    warning_log.file = &shared_log_file;
    debug_log.file = &shared_log_file;
    
    // But each has different settings
    error_log.log_level = LPD_LOG_ERROR;
    warning_log.log_level = LPD_LOG_WARN;
    debug_log.log_level = LPD_LOG_DEBUG;
}
```

### Visual Representation:

```

error_log ----\
               \
warning_log ----+---> shared_log_file (one actual file)
               /
debug_log ----/
```

### Why This is Useful:

#### 1. **Resource Efficiency**

- Only **one file descriptor** needed
- Only **one file buffer** in memory
- Reduces system resource usage

#### 2. **Centralized Output**

- All log messages go to **one location**
- Easier to read/analyze logs
- No need to merge multiple files

#### 3. **Different Log Levels, Same Destination**

- Each log can have different `log_level` filtering
- But all write to the same physical file
- Clean separation of concerns

#### 4. **Thread Safety Benefits**

- File locking/synchronization happens at **one place**
- Multiple threads can use different log structs
- But file access is coordinated

This is a **very common pattern** in logging systems - multiple loggers, shared destinations!

### `LPD_LOG_NOTICE`

- **Severity**: (6) represents a level of importance higher than informational messages () but lower than warnings (). `LPD_LOG_NOTICE``LPD_LOG_INFO``LPD_LOG_WARNING`
- **Use**: At this level, the logs will usually record significant but non-critical events to provide insight into the application's behavior.

#### Practical Implications of Setting : `lpd_log.log_level = LPD_LOG_NOTICE`

1. **Logs with lower severity will be ignored**: Only messages with `log_level >= LPD_LOG_NOTICE` (i.e., , , , etc.) will be processed by the logging system. `LPD_LOG_NOTICE``LPD_LOG_WARNING``LPD_LOG_ERR`
    
2. **Typical Use Case**:
    
    - Logging events that are important for monitoring or tracking the application's behavior.
    - Monitoring notable actions such as routine system operations or configuration updates.

####  [`lpd_log_t` ]

- **Definition:** `src/core/ngx_log.h` and `ngx_log.c`
    
- **Purpose:** All logging throughout NGINX (errors, info, debugging).

###  [ Print Functions for Strings.c ]

Here are the string-related functions critical to NGINX, especially for logging (like ngx_sprintf_str and its family):

#### Most Important NGINX String Functions for Logging:

1. **ngx_sprintf / ngx_snprintf / ngx_slprintf / ngx_vslprintf**
   - These are NGINX's custom "safe" string formatting functions.
   - Used everywhere for formatting log messages, error strings, etc.
   - Example: In `src/core/ngx_log.c` and log modules, these functions format the actual log lines.

2. **ngx_strcasecmp / ngx_strncasecmp / ngx_strncmp**
   - Used to compare strings, often for log format names or configuration parsing.

3. **ngx_strstrn / ngx_strcasestrn / ngx_strlcasestrn / ngx_strnstr**
   - Used for substring searching, helpful in parsing config and filtering logs.

4. **ngx_strlow / ngx_strnlen**
   - Utility functions for lower-casing and length calculation; useful in config and log handling.

5. **ngx_error_log / ngx_log_error / ngx_log_debug_core**
   - These are the core error/logging functions, which internally use the string formatting functions above.

6. **ngx_parse_size / ngx_atoi / ngx_atofp / ngx_hextoi**
   - Used for converting strings in log configuration (like buffer sizes, etc.).

7. **ngx_hex_dump**
   - Used to format binary data (like request/response bodies) for logging.

8. **ngx_conf_log_error**
   - Used for logging configuration errors; relies heavily on string formatting.

#### Where They're Used:
- **Core Logging:** Formatting the actual log lines (`ngx_log.c`, `ngx_http_log_module.c`, `ngx_stream_log_module.c`)
- **Log Format Compilation:** Parsing and matching log formats by name uses string comparison functions.
- **Configuration Parsing:** Reading and validating config values for logs involves string conversion/comparison.
- **Log Variable Expansion:** Expanding variables in log formats relies on string manipulation.

### Why NGINX Uses "Custom" Functions:
- NGINX's own functions (instead of standard C library) are designed for performance, buffer safety, and sometimes platform portability.

---

**Summary:**  
Critical string functions for NGINX logging include the custom formatting family (`ngx_sprintf`, etc.), string comparison/search utilities, and conversion helpers. These are used throughout logging code to format, compare, and parse strings in logs and configurations.
# os

## lpd_epoll_module.c

// lapideus_epoll.c

#include <sys/epoll.h>
#include <sys/eventfd.h>
#include <fcntl.h>
#include <errno.h>

/`/ File-scope state`
`static int                lpd_epoll_fd = -1;`
`static struct epoll_event *lpd_ep_events = NULL;`
`static unsigned           lpd_nevents = 0;`
`static int                lpd_notify_fd = -1;  // optional eventfd for wakeups`

`static inline int set_cloexec(int fd) {`
    `return fcntl(fd, F_SETFD, FD_CLOEXEC);`
`}`

`lpd_int_t lpd_epoll_init(lpd_cycle_t *cycle, lpd_msec_t timer /*unused*/) {`
    `lpd_epoll_conf_t *epcf = lpd_event_get_conf(cycle->conf_ctx, lpd_epoll_module);`
    `if (!epcf || epcf->events == 0) {`
        `lpd_log_error_core(LPD_LOG_EMERG, cycle->log, 0, "epoll: invalid config");`
        `return LPD_ERROR;`
    `}`

    `// 1) Create epoll instance once`
    `if (lpd_epoll_fd == -1) {`
        `lpd_epoll_fd = epoll_create1(EPOLL_CLOEXEC);`
        `if (lpd_epoll_fd == -1) {`
            `lpd_log_error_core(LPD_LOG_EMERG, cycle->log, errno, "epoll_create1() failed");`
            `return LPD_ERROR;`
        `}`

        `// Optional: eventfd for cross-thread/process wakeups`
        `lpd_notify_fd = eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);`
        `if (lpd_notify_fd != -1) {`
            struct epoll_event ev = { .events = EPOLLIN, .data.u64 = LPD_EPDATA_NOTIFY };
            if (epoll_ctl(lpd_epoll_fd, EPOLL_CTL_ADD, lpd_notify_fd, &ev) == -1) {
                lpd_log_error_core(LPD_LOG_WARN, cycle->log, errno, "epoll: eventfd add failed; disabling notify");
                close(lpd_notify_fd);
                lpd = -1;
            }
        } else {
            lpd_log_error_core(LPD_LOG_DEBUG, cycle->log, errno, "epoll: eventfd not available");
        }

        // Optional: test EPOLLRDHUP support once (set a global flag if you care)
        // lpd_test_epoll_rdhup(cycle);
    }

    // 2) (Re)size events buffer if needed
    if (lpd_nevents < epcf->events) {
        void *newbuf = realloc(lpd_ep_events, sizeof(struct epoll_event) * epcf->events);
        if (!newbuf) {
            lpd_log_error_core(LPD_LOG_EMERG, cycle->log, errno, "epoll: events buffer alloc failed");
            return LPD_ERROR;
        }
        lpd_ep_events = newbuf;
        lpd_nevents   = epcf->events;
    }

    // 3) Activate OS I/O and epoll action vtables
    lpd_io            = lpd_os_io;
    lpd_event_actions = lpd_epoll_module_ctx.actions;

    // 4) Describe loop semantics
`#if defined(LPD_HAVE_CLEAR_EVENT)`
    `lpd_event_flags = LPD_USE_CLEAR_EVENT`
`#else`
    `lpd_event_flags = LPD_USE_LEVEL_EVENT`
`#endif`
                    `| LPD_USE_GREEDY_EVENT`
                    `| LPD_USE_EPOLL_EVENT;`

    return LPD_OK;
}
``


## **`lpd_files.h`**

- **`LPD_INVALID_FILE`**  
    Represents an invalid file descriptor (commonly `-1`). Used to indicate errors or an unopened file.
    
- **`LPD_FILE_RDONLY` / `LPD_FILE_WRONLY` / `LPD_FILE_RDWR`**  
    Shorthand names for the file access modes: read-only, write-only, and read/write.  
    They map to standard POSIX flags: `O_RDONLY`, `O_WRONLY`, and `O_RDWR`.
    
- **`LPD_FILE_APPEND`**  
    Tells the file operations to append data to the end of the file rather than overwrite. (`O_APPEND`)
    
- **`LPD_FILE_CREAT`**  
    Create the file if it doesn't exist. (`O_CREAT`)
    
- **`LPD_FILE_TRUNC`**  
    Truncate the file to zero length if it already exists (clearing its contents). (`O_TRUNC`)
    
- **`LPD_FILE_EXCL`**  
    When used with `O_CREAT`, it will fail if the file already exists (prevents overwriting). (`O_EXCL`)
    
- **`LPD_FILE_NOCTTY`**  
    Prevents the file from becoming the controlling terminal for the process. (`O_NOCTTY`)
    
- **`LPD_FILE_NONBLOCK`**  
    Opens the file in non-blocking mode (operations return immediately, do not wait). (`O_NONBLOCK`)
    
- **`LPD_FILE_SYNC`**  
    Writes are synchronized to disk; operations only return after the data is physically written. (`O_SYNC`)

#### **`lpd_open_file(const char* path, int flags, int mode)`**  
Opens a file at the specified path with the given flags (such as read, write, create, etc.) and mode (file permissions if creating).  **Returns:** A file descriptor, or -1 on error.
    
#### **`lpd_read_file(lpd_fd_t fd, void* buf, size_t size)`**  
Reads up to `size` bytes from an open file (identified by `fd`) into the buffer `buf`.  
**Returns:** The number of bytes read, or -1 on error.
    
#### `**lpd_write_file(lpd_fd_t fd, const void* buf, size_t size)**`  
Writes up to `size` bytes from the buffer `buf` to the open file identified by `fd`.  
    **Returns:** The number of bytes written, or -1 on error.
    
#### `**lpd_close_file(lpd_fd_t fd)**`  
Closes the file associated with the file descriptor `fd`.  
    **Returns:** 0 on success, or -1 on error.

## `lpd_files.c`


would nginx have const u_char* as the type for the buf in the read file function?

`ssize_t  lpd_read_file(lpd_file_t* file`, ==u_char* buf,== `size_t size, off_t offset, lpd_log_t* log);`  

`ssize_t  lpd_write_file(lpd_file_t* file`, ==**const u_char* buf**==, `size_t size, off_t offset, lpd_log_t* log);`

No, in NGINX (and in conventional POSIX/C practice), the buffer type for a file **read** function is typically a **non-const pointer** (`u_char* buf`), not `const u_char*`.

**Reason:**

- A read operation writes data into the buffer provided by the caller. The function is expected to modify the content at `buf`.
- Declaring `const u_char* buf` would incorrectly indicate that the contents of `buf` are not meant to be changed, which would not match the function’s contract.

**Write functions** take `const u_char* buf` because the buffer contents will not be modified by the function; they are just copied/written out.

---

---

### Reference (NGINX style example)

`ssize_t ngx_read_file(ngx_file_t *file, u_char *buf, size_t size, off_t offset);`  


- For reading: `u_char* buf`
- For writing: `const u_char* buf`

### Winsock

Winsock is an API for network communication on windows. allows programs on windows to send and receive data over networks using protocols like TCP/IP

## Server.h

### Server Constructor

[ Variables ]

int domain
int service 
int protocol
int backlog

struct sockaddr_in address;

int socket;
void (*launch)(struct Server *server);

### Declaration of Server Object

struct Server server_constructor(int domain, int service, int protocol, u_long net_interface, int port,            int backlog, void (*launch)(struct Server *server));


## Server.c

### Line 30: server.address.sin_family = domain;

1. **`server.address`**
    
    - `server` is a struct that likely contains a `sockaddr_in` field named `address`.
    - `address` stores information about the socket’s address.
2. **`sin_family`**
    
    - A field in `struct sockaddr_in` that specifies the address family (IPv4 or IPv6).
    - It determines what type of addresses the socket can use.
3. **`domain`**
    
    - This should be a value like `AF_INET` (for IPv4) or `AF_INET6` (for IPv6).
    - If `domain` is a variable, it should be assigned to one of these values 

## testServer.c

### Line 20: int new_socket = accept(server->socket, (struct sockaddr *)&server->address, (socklen_t *)&address_length);

- **`accept()` waits for a client** to connect and returns a new socket (`new_socket`) to communicate with that client.
- The original server socket (`server->socket`) **only listens** for connections, while `new_socket` is used **for sending/receiving data**.

🔹 **Parameter breakdown:**

- `server->socket` → The listening socket.
- `(struct sockaddr *)&server->address` → Stores client address details.
- `(socklen_t *)&address_length` → Passes the size of `server->address` to `accept()`.

🔹 **What happens?**

- If a client connects, `accept()` returns a new socket descriptor (`new_socket`).
- If it fails, it returns `-1` and sets `errno` (e.g., connection issues).

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);

- `sockfd`: The server socket.
- `addr`: Pointer to a `struct sockaddr` to store the client's address.
- `addrlen`: Pointer to a `socklen_t` variable that holds the size of `addr`.






## string_util.h

### strdup_http

char* strdup_http(const char* string, unsigned int max_length)  
{  
    if (string == NULL) return NULL;  
  
    for ( unsigned int i = 0; i < max_length; i++)  
    {        if (string[i] == '\r' && string[i + 1] == '\n' && string[i + 2] == '\r' && string[i + 3] == '\n')  
        {            unsigned int length = i + 4; // include the \r\n\r\n  
            char* copy = (char*)malloc(length + 1);  
            if (copy == NULL) return NULL;  
  
            memcpy(copy, string, length);  
            copy[length] = '\0';  
            return copy;  
        }    }    return NULL;  
}


## HTTP Parser

When working with stdup_http I need to pass the byte amount from recv() into the HTTP_request Constructor so that I can read the proper amount of bytes


## lpd_connection_t

- **Definition:** `src/core/ngx_connection.h` and `ngx_connection.c`
    
- **Purpose:** Describes a single TCP connection (socket, state, events, etc.)
    
- **References:** Used in core and all protocol modules (HTTP, mail, stream, etc.).



## lpd_event_t

- **Definition:** `src/event/ngx_event.h` and `ngx_event.c`
    
- **Purpose:** Encapsulates read/write events for each connection; tracks event state, handlers, timeouts, etc.
    
- **References:** Used in event modules and by the connection layer.

## (Optional) ngx_event_module_t

- **Definition:** `src/event/ngx_event_module.h` and corresponding files for epoll, select, kqueue, etc.
    
- **Purpose:** Abstracts out event backend (epoll/kqueue/etc.) behind a uniform interface.
    



## Priority Implementation Order

**Phase 1 (Essential):**

1. `lpd_log_error` and `lpd_log_debug`
2. Basic formatting functions (`lpd_sprintf`, `lpd_snprintf`)
3. File writing functions
4. Memory pool basics

**Phase 2 (Important):**

5. Time formatting functions
6. File management (open, close, reopen)
7. Log level management
8. Cleanup functions

**Phase 3 (Advanced):**

9. Configuration parsing
10. Cycle management
11. Advanced file operations
12. Performance optimization functions

This set of functions will give you nginx-like logging capabilities with proper memory management, file handling, formatting, and configuration support. Start with Phase 1 functions for basic functionality, then expand to Phases 2 and 3 for full-featured logging.


### lpd_gcc_atomic_pcc.h

The lpd_atomic_fetch_add(lpd_atomic_t* value, lpd-atomic_t* add) function is an atomic function that increments the value parameter by the add parameter using inline assembly