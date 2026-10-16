Related to [[Lapideus]], [[Linux]] , [[Lapideus Documentation]] [[Nagles Algorithm]]
#lapideus_build_log
## Graceful Exit
03.22.25

This was added to the testServer.C File.
You need to implement a **graceful exit** to properly handle termination signals (like `Ctrl+C`) and ensure cleanup of resources such as sockets and Winsock before exiting.

### Steps to Implement a Graceful Exit:

1. **Handle Termination Signals** (`SIGINT` for `Ctrl+C`, `SIGTERM` for system termination).
2. **Set a Global Flag (`keep_running`)** to signal the server to stop.
3. **Modify the `while(1)` loop** in `launch()` to check `keep_running`.
4. **Ensure Proper Cleanup** (close sockets and call `WSACleanup()`).

### Updated Code:

Modify your program as follows:

#### 1. Add a Signal Handler:

```c
void handle_signal(int signal)
{
    printf("\nReceived termination signal (%d). Shutting down...\n", signal);
    keep_running = 0; // Stop the server loop
    closesocket(new_socket); // Close any open client socket
}
```

#### 2. Register the Signal Handler in `main()`:

```c
signal(SIGINT, handle_signal);
signal(SIGTERM, handle_signal);
```

#### 3. Modify `launch()` to Exit Gracefully:

Replace the infinite `while (1)` loop with:

```c
while (keep_running)
```

#### 4. Ensure Cleanup in `main()`:

After `server.launch(&server);`, close the server socket before `WSACleanup()`:

```c
closesocket(server.socket);
```

---


- Press **Ctrl+C** → `handle_signal()` sets `keep_running = 0`, breaking the loop.
- `accept()` checks if `keep_running` is `0` and exits cleanly.
- The server socket and Winsock are cleaned up before exiting.

This ensures your server shuts down **gracefully** without leaving open connections or resources.

### 03.25.25

- Implemented the graceful exit using signal()
- Apparently signal() is a more generic function which is recognized on UNIX systems as well and I should look to use ctrlHandler() if I looking for a more Windows based implementation of the WebServer


## MultiThreading and MultiProcessing

### 03.26.25

Using POSIX ,(Portable Operating System Interface), is a **set of standards** or an **API specification** that defines a standard operating system interface and environment for software compatibility across different UNIX-like operating systems, The plan is to implement a system that brings in unique threads for respective client connections so that they can run concurrently.

- my PC CPU is a Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz   3.70 GHz
	This CPU specs:
		Physical Cores - 6 cores
		Logical Cores(threads) - 12 threads 
	 - With 6 Physical cores my cpu can run 6 independent processes
	 - Additionally, with Hyper-Threading each core can handle two simultaneous instruction streams (threads), this is equivalent to 12 logical threads
	 - I can run 12 concurrent server threads before the CPU becomes oversaturated

- With **multithreading** → You have **one process** with many threads. Even if they run in parallel on multiple cores, they still share memory.
    
- With **multiprocessing** → You have **multiple processes** that each run separately, with their own memory space.

## Parsing HTTP Requests

### 03.28.25

- Create a struct that holds the various pieces of the HTTP Request

### 04.09.25

**Today, I learned how to integrate a C library by using multiple C files under one header file. Additionally, I explored replacing various functions from the `<string.h>` library, as some of them are prone to vulnerabilities. As a result, I am implementing my own versions of these functions. For exam ple, I am writing a custom `string_copy` function to replace `strdup` and implementing my own `strtok` function.** 

**The standard `strtok` function is problematic because it uses an internal static variable, which persists outside of the function's scope. This makes it non-thread-safe and unsuitable for concurrent programming. Additionally, `strtok` lacks proper bounds checking, which can lead to potential buffer overflows. The strdup function also does not have proper bounds checking which can lead to buffer overflows.** 

I've discovered that because much of the C libraries were developed so long ago, in the 1980's, they were developed more for functionality rather than security. Additionally, security in a more general sense was not something that was a common focus for developers until later years. The libraries were developed with the intention that programmers would implement their own checks on the libraries and called for more attention to such matters. 

The **C standard libraries** have **evolved** over the years, but many of the core functions have remained the same due to the need for **backward compatibility**. Changing or enhancing existing functions could break **legacy code** that has relied on the original behavior of these functions for decades.

### 04.12.25

Today I learned how threads interact with memory. Each thread gets its own stack, while all threads share the heap. However, even though the heap is shared, each thread can allocate its own independent memory on it. A thread can be thought of as a single sequential set of instructions operating with its own stack and access to the shared heap.

I implemented the strdup_http function which really just is a function to copy out strings into a new copy with proper bounds checking specified by the length argument. Currently I am working on the strtok_safe which is a tokenizer that is safer than str_tok as it doesn't hold a static variable and is compatible with multithreading. Additionally strtok_safe doesn't pollute the original string that is passed in and returns the tokens with the original string untouched. I am deciphering the implementation of str_tok and am looking to implement it into the http_request file so that I can parse out the various components of the http request into the httprequest struct. 

## 04.13.25

Today I dove into pointers and double pointers. Double pointers have the ability if passed into a function as an argument to mutate variables outside of the function scope. Additionally, I looked into the structure of the heap and stack in memory and discovered that they take on a unique form in how they are able to co-exist. The heap starts allocating from memory addresses right after the data segment and continues to allocate higher and higher memory addresses until the heap is full. If more memory is necessary for the heap it will use memory in the free space if available.
The stack starts allocating at the highest available memory address and allocates addresses downwards. Similar to the heap, if more memory is needed for the stack, memory will be allocated from the free space if available. 

## 04.16.25

Although previously worked on and understood, I thought it best to include a refresher of cmake: Due to the fact that C requires you to explicitly link files together and specify file compilation that cmake with it being a build automation tool, allows you to manage your project in a more organized manner compared to other languages like java where most of the compilation of files is handled by the internal package manager. Today I integrated the custom string functions I wrote into the parsing logic of the HTTPRequest.c File. I need to include more error handling as I don't want HTTP requests to be assumed to be properly formatted. 

## 04.24.25

Amidst the parsing of the http request I have discovered that there is a faster and more efficient way of parsing the request. Currently the main server loop passes the raw bytes of the http request into the core function of HTTPRequest.c which originally was intended to parse out the start line and the headers by creating one large copy and then duplicating each variable(Method, URI, HTTPVersion) from a tokenizer. However, after some research I have discovered by looking into some other web server implementations that pool allocators are much more efficient. Instead of allocating memory individually for each component, a pool allocator performs one large allocation and hands out chunks of that memory as needed — significantly reducing allocation overhead and improving memory locality. 

## 04.25.25

The memory pool allocator that I will incorporate will be able to exceed the performance of multiple malloc calls since the memory pool allocator only needs one large single malloc call with other features such as 8 byte alignment, free the entire pool when done, and allocates sequentially. 

## 05.20.25

I've uncovered a design pattern that optimizes HTTP request handling by eliminating the need to scan for null terminators. Instead of relying on null-terminated strings, I will represent each token in the request (method, URI, HTTP version, headers, etc.) using a pointer and an explicit length. This avoids `strlen()` calls and enables binary-safe, zero-copy parsing.

The process begins by reading the raw HTTP request into a buffer. Then, each component is tokenized by identifying its start and end boundaries — without modifying the buffer. These tokens are assigned as pointers within the buffer and wrapped in lightweight structs that track both their position and length.

Allocations for these structs are managed using a memory pool, ensuring that memory is efficiently grouped and deallocated in bulk when the request ends. Finally, all tokenized components are composed into a higher-level `HTTP_Request` struct, which provides structured access to the parsed fields throughout the lifecycle of the request.

## 06.11.25

So far I have a strong understanding of the memory pool allocator and understand why it is implemented for this project. I am currently looking to implement the parsing of the http request. Currently I have the http request struct setup to store the request and also have logic to parse the start line. Today I added the beginning logic to parse the http request headers and am looking to finish tomorrow. The current http parser is merely mapping out the request and storing each of the pieces in pointers. Now after the http request is parsed the logic will be needed to process the request and also connect the state machines to setup a state based concurrency model.

06.29.25

I have discovered a new direction for the project. This is the vision. I want to use burpsuit to send requests to nginx then give them to the c web server and then the external proxy and then to the website. I want to configure the web server to run with a ai assistant connected run entirely offline airgapped and be able to act for purely observational reasons. 

## 07.28.225

Today I am working on lpd_log.c. I am looking into establishing the lpd_log_t struct and writing some function implementation. I have come across variables such as: typedef uintptr_t lpd_atomic_uint_t; 
- Atomic operations are CPU-level instructions that allow safe updates to shared variables between threads, eliminating race conditions without using blocking locks. 
- Mutex's on the other hand are blocking by nature. when a thread calls pthread_mutex_lock it will pause execution if another thread already holds a lock. It won't continue until: the other thread releases the lock with pthread_mutex_lock(&lock); And the current thread acquires the lock: A **mutex is a lock** — it's used to make sure only one thread can access a piece of code or data at a time. If another thread tries to enter while it's locked, it waits until the mutex is released

`pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;`
`void *thread_func(void *arg) {`
    `pthread_mutex_lock(&lock);   // If already locked, thread will block here`
    `// Critical section`
    `pthread_mutex_unlock(&lock);`
`}`

so essentially I am using lpd_atomic_uint_t       connections; as an unsigned long variable that avoids race conditions

### 09.12.25

- Kqueue is for Free BSD, Open BSD, Net BSD, and MacOS

In event_loop.h
lpd_event_actions_t which is a struct holding the function pointers to the Epoll module functions. This lpd_event_actions_t name is given a #define lpd_event_actions_t lpd_event_actions for simpler use. Additionally the functions within the lpd_actions


### 09.16.25

Today I am going over NGINX's use of LRU queue's. LRU queues (Least-recently used queue) is an ordering of items so that you can quickly evict the item that hasn't been touched in the longest time. Every time you use an item you promote the item to the front and the item at the back is the least-recently used (LRU) and the one you evict when you have need for space. These use O(1) speed.

### 02.11.26

Today I am working on the `lpd_event_accept.c` file. There is a flag under the if LPD_HAVE_UNIX_DOMAIN, that is called `tcp_nopush` and `tcp_nodelay` that act as a way to allow better throughput for larger file transfers  for static files over the network. tcp_nopush corresponds to disabling Nagle's algorithm (send small writes immediately). Conceptually:

- **`tcp_nodelay`**: “send small stuff _now_”
- **`tcp_nopush`**: “wait a bit so we can send a _bigger_ chunk”

They can work against each other, so Nginx typically **toggles** them depending on what it’s currently sending (headers vs streaming vs buffered output).
## 06.13.26

### AI Projections

I appreciate the detailed context. Let me be candid: I think your self-assessment is **optimistic**. The cycle, timers, and server controller are genuinely incomplete, but they are not the *only* things standing between you and 100k concurrent connections. There are several other load-bearing pieces that are stubbed, half-wired, or contain bugs that will prevent the server from even running, let alone scaling.

Let me verify a few things against the actual codebase rather than relying solely on the attachment, so I can give you an accurate picture.The epoll module declares many functions in its actions struct but only `lpd_epoll_init` is defined. Let me verify which event-action functions actually have bodies, and check the listening socket setup and worker process layer.Let me check a few more critical pieces to give an accurate assessment.Let me check a few more things to confirm there's no defined `lpd_event_actions` instance and what the listening-socket/bind setup looks like.I've explored the codebase thoroughly. Let me give you an honest, grounded assessment, because the situation is a fair bit more involved than "just three files left."

#### Short answer

No — that's an over‑optimistic assessment. The three areas you mentioned (cycle, timers, server controller) are genuinely incomplete, but several **other load‑bearing pieces that sit directly on the 100k‑connection hot path are also unimplemented or stubbed**. You're closer architecturally than functionally; the skeleton is good (very nginx‑like), but the parts that actually move bytes and dispatch events aren't wired up yet.

Let me break down what I found.

#### What you correctly identified as incomplete

**1. Cycle (`lpd_cycle.c`)**
- `lpd_init_connections()` is a no‑op that returns an uninitialized variable:
```c++
static lpd_int_t lpd_init_connections(struct lpd_cycle_s *cycle) {
      lpd_int_t i;
      return i;   // returns garbage
  }
```

- The entire real implementation (connection pool, free list, master/worker loop, pidfile) is commented out, and that commented version references a different struct layout (`c->next`, `cycle->connections` as a free list) than your actual `lpd_cycle_s` (which uses `free_connections` + `read`/`write` event pairs). So it can't just be uncommented.
- `lpd_cycle` is declared `volatile struct lpd_cycle_s*` here but `extern volatile lpd_cycle_t*` in the header — consistent, but the global is defined in a function body region that's only partly live.

**2. Timers (`lpd_event_timer.c` / `.h`)**
- `lpd_event_find_timer()` is implemented, but **`lpd_event_expire_timers()` is declared and called (in `lpd_event.c`) but never defined.** That's a link error and the core of timeout handling.
- `lpd_event_add_timer()` (in the header) references `LPD_TIMER_LAZY_DELAY` and `lpd_abs()`, neither of which I can find defined anywhere — they'll fail to compile the moment that inline is used.
- `lpd_times.c`: `lpd_time_update()` is an **empty body**, and `lpd_gmtime()` is declared but never defined. Since timers key off `lpd_current_msec`, and nothing ever updates it, every timer computation is against a frozen clock.

**3. Server controller** — correct, there is effectively no Unix server controller. `psx_server.c` is empty, and `lapideus.c`'s `core_main()` just `return 0`. There's no listen‑socket setup, no worker spawn, no event‑loop entry on Unix.

#### What you missed — and these are on the critical path

**4. The epoll module is half‑built (`lpd_epoll_module.c`)**
This is the single most important file for your goal and it's the most incomplete of the event files:
- Only `lpd_epoll_init()` is defined.
- `lpd_epoll_add_event`, `lpd_epoll_del_event`, `lpd_epoll_add_connection`, `lpd_epoll_del_connection`, `lpd_epoll_process_events`, `lpd_epoll_done`, and `lpd_epoll_notify` are all **declared and placed in the `lpd_event_actions_t` struct but never defined.** There is no `epoll_wait` call anywhere in the project. Without `process_events`, there is no event loop at all.

**5. The OS I/O layer is entirely unimplemented (`lpd_socket.c`, `psx_server.c`)**
- `lpd_socket.c` is empty.
- `lpd_unix_recv`, `lpd_unix_send`, `lpd_readv_chain`, `lpd_writev_chain` are declared in `lpd_os.h` but **none are defined.** Your accept path assigns `c->recv = lpd_recv` (a macro to `lpd_os_io.recv`), but `lpd_os_io` itself is **never defined or populated** anywhere. So even after accept, a connection has no working read/write.

**6. Buffers and queues are stubs**
- `lpd_buf.c` is empty; `lpd_buf.h` has a struct‑name mismatch (`lpd_buf_s` forward‑declared but `lpd_buf_t` defined as a separate tag), so chains can't compile cleanly.
- `lpd_queue.c` is empty (the macros are header‑only, which is fine), but note `lpd_queue_insert_tail` is currently identical to `insert_head` — a copy/paste bug that will corrupt your reusable‑connection and posted‑event lists at scale.

**7. Connection lifecycle bugs that surface under load**
In `lpd_close_connection()`, `lpd_free_connection(connection)` is called **before** reading `connection->fd`, and `fd` is captured after the connection has been returned to the free list — a use‑after‑recycle that will bite hard at high churn. Also `lpd_drain_connections` has a loop that `break`s without using `q`, then dereferences past it.

**8. No `main`/worker/process model**
There's no `fork`, no worker process management, no `setrlimit(RLIMIT_NOFILE)` raise. 100k concurrent connections requires lifting the FD limit and (realistically) `SO_REUSEPORT` workers — none of that exists yet.

#### Honest verdict

Your **design** is close to nginx‑grade and the parsing, rbtree, pool allocator, and accept logic are largely done. But the chain from "accept a connection" → "register with epoll" → "wait for events" → "read/write bytes" → "expire timers" has **at least four missing links** (epoll actions, `lpd_os_io` + recv/send, `lpd_event_expire_timers`, `lpd_time_update`), plus the cycle/controller you already named.

So the realistic remaining work, in dependency order:

1. `lpd_time_update()` + `lpd_current_msec` updating (timers depend on it)
2. `lpd_event_expire_timers()` + the missing `LPD_TIMER_LAZY_DELAY`/`lpd_abs` macros
3. Unix I/O: define `lpd_os_io`, implement `lpd_unix_recv`/`lpd_unix_send`
4. Finish the epoll module (especially `process_events` with `epoll_wait`)
5. Fix `lpd_queue_insert_tail` and the `lpd_close_connection` use‑after‑free
6. Real `lpd_init_connections()` matching your current struct
7. Server controller: listen sockets, `setrlimit`, worker loop, `main`

Would you like me to start at the top of that list? I'd suggest beginning with `lpd_time_update()` and `lpd_event_expire_timers()`, since the event loop and timeouts can't function without them, and then move to the epoll `process_events` implementation.