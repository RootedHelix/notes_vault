
## 05.28

## 03.28.25
I have spent the past few months familiarizing myself with the c language. I completed a course through w3schools to learn the basic syntax in addition to starting and working on a project to build my own webserver. I have thus far followed along with youtube tutorials to construct a working webserver that is able to establish a connection between the server and the network and the client. Additionally, I have enabled functionality of what parsing out an httprequest should generally look like. 

The plan from here on out is to no longer take from the designs of the tutorials but rather use them as inspiration for my own project. Now that I have a clear image of what these concepts look like in application I can now write them and claim them in my own implementation. The remaining pieces of knowledge that I still need to ascertain from these tutorials are as follows:

- Gain a significant understanding of data structures such as: Nodes, Linked Lists, queues, dictionaries, and binary trees.

Each day that passes I will complete an additional data structure to strengthen my understanding of these topics. 

Furthermore, I will complete a full implementation of the system to the extent of what the tutorials introduce to me.

Following these steps I will take the knowledge and implement my own design to the base system.

Finally, I will expand on the system I create which by that point should match the extent of the system constructed in the tutorials. I will add features listed in the /C Basics/C WebServer and /C WebServer/Lapideus_Direction notes in obsidian. 



## Microservice Architecture
Your setup is actually a simple **microservices** pattern:

- **Web Service** (NGINX) - Handles HTTP concerns
- **Application Service** (Your C Server) - Handles business concerns

- run the server behind nginx and a cdn. Use application server as a cache for cdn, in case of cache misses on the cdn




- run the server behind nginx and a cdn. Use application server as a cache for cdn, in case of cache m



- **Database Service** (PostgreSQL/MySQL/etc.) - Handles data concerns

This is **exactly** how companies like Netflix, Amazon, Google structure their applications.


## Fluency Comes at the End of building the Project


# File Serving

Got it — here are the **exact NGINX source files** you’d copy concepts from to build a **minimal, equal-performance static file path** (hot files, zero-copy, minimal syscalls). I’ve separated **required** vs **optional**.

# Required (for perf-parity on hot static files)

- `src/http/modules/ngx_http_static_module.c`  
    The static content handler (URI → filesystem path, perms, handoff to send). ([FreeNginx](https://freenginx.org/hg/nginx/file/19251fb83187/src/http/modules/ngx_http_static_module.c?utm_source=chatgpt.com "19251fb83187 src/http/modules/ngx_http_static_module.c"))
    
- `src/core/ngx_open_file_cache.c` and `src/core/ngx_open_file_cache.h`  
    Open-file & stat/metadata cache (avoids repeat `open()`/`stat()` on hot files). ([Arch Linux](https://archlinux.org/packages/extra/x86_64/nginx-src/files/?utm_source=chatgpt.com "nginx-src 1.28.0-4 (x86_64) - File List"))
    
- `src/core/ngx_output_chain.c`  
    The output chain that glues header buffers + file buffers and feeds the OS writer. ([Agentzh](https://agentzh.org/misc/code/nginx/core/ngx_output_chain.c.html?utm_source=chatgpt.com "src/core/ngx_output_chain.c - nginx-1.7.10"))
    
- `src/os/unix/ngx_linux_sendfile_chain.c`  
    Linux zero-copy writer using `sendfile(2)` (nonblocking, partial-send handling). ([FreeNginx](https://freenginx.org/hg/nginx/file/c901f2764c27/src/os/unix/ngx_linux_sendfile_chain.c?utm_source=chatgpt.com "c901f2764c27 src/os/unix/ngx_linux_sendfile_chain.c"))
    
- `src/http/ngx_http_header_filter_module.c`  
    Emits the actual HTTP headers (Content-Length, Date, Server, etc.) before the body. ([Debian Sources](https://sources.debian.org/src/nginx/1.22.1-9%2Bdeb12u2/src/http/ngx_http_header_filter_module.c/?utm_source=chatgpt.com "File: ngx_http_header_filter_module.c"))
    

> Why these five?  
> Static handler (routing) → open-file cache (syscall cuts) → output chain (buffer plumbing) → OS sendfile path (zero-copy) → header filter (HTTP correctness). That’s the lean critical path NGINX uses for raw static-file throughput.

# Optional (small, high-value adds)

- **304 / conditional handling** (bandwidth saver, cheap once ETag/mtime exist):  
    `src/http/modules/ngx_http_not_modified_filter_module.c`. (Module name and role; NGINX docs show the related controls live in the core HTTP module.) ([Nginx](https://nginx.org/en/docs/http/ngx_http_core_module.html?utm_source=chatgpt.com "Module ngx_http_core_module"))
    
- **Range / 206 support** (videos, resumes):  
    `src/http/modules/ngx_http_range_filter_module.c`. ([Fossies](https://fossies.org/dox/nginx-1.29.1/ngx__http__range__filter__module_8c_source.html?utm_source=chatgpt.com "nginx: src/http/modules/ngx_http_range_filter_module.c ..."))
    
- **MIME map** (correct `Content-Type`):  
    Config file `conf/mime.types` included by `http { include mime.types; }`. ([Nginx](https://nginx.org/en/docs/http/ngx_http_core_module.html?utm_source=chatgpt.com "Module ngx_http_core_module"))
    

If you mirror just the **Required** set in your Lapideus layout (static handler → open-file cache → output chain → `sendfile` writer → header filter), you’ll have the same fundamentals NGINX uses to hit top-tier static-file performance.

# Remaining Pieces for Nginx level design

With those nginx-style static file serving components implemented, you'd have a **highly optimized static file server** that could theoretically handle 100k connections, but you'd still be missing several critical pieces for **actual 100k concurrent connection handling**:

## What You'd Have (High-Performance Static Serving)

- Zero-copy file delivery (sendfile)
- Efficient file caching and metadata caching
- Proper HTTP response handling
- Range requests, conditional responses
- Non-blocking I/O foundation

## What You'd Still Need for 100K Connections

### 1. **Advanced Memory Management**

```

// Current: Basic pool allocator// Need: Zero-allocation hot paths, object poolingsrc/core/├── lpd_slab_allocator.c/.h         // Fixed-size object pools├── lpd_buffer_pool.c/.h            // Buffer recycling system├── lpd_connection_pool.c/.h        // Connection object reuse
```

### 2. **Epoll/Kqueue Event Scalability**

```

// Current: Basic event system// Need: Edge-triggered, batch processingevent/modules/├── lpd_epoll_module.c/.h           // Linux epoll (ET mode)├── lpd_kqueue_module.c/.h          // BSD/macOS kqueue├── lpd_devpoll_module.c/.h         // Solaris /dev/poll├── lpd_event_batch.c/.h            // Batch event processing
```

### 3. **Connection State Management**

```

// Need: Efficient connection lifecyclecore/├── lpd_connection_manager.c/.h     // Connection pool & reuse├── lpd_keepalive.c/.h             // HTTP keep-alive management├── lpd_timeout_manager.c/.h        // Timeout handling at scale
```

### 4. **Advanced HTTP Pipeline**

```

// Need: Pipelined request handlinghttp/├── lpd_http_pipeline.c/.h          // HTTP/1.1 pipelining├── lpd_http_request_pool.c/.h      // Request object pooling├── lpd_http_parse_optimize.c/.h    // Zero-copy header parsing
```

### 5. **System Resource Optimization**

```

// Need: OS-level optimizationsos/unix/├── lpd_setrlimit.c/.h             // Resource limit management├── lpd_numa.c/.h                  // NUMA-aware allocation├── lpd_cpu_affinity.c/.h          // CPU pinning├── lpd_tcp_optimize.c/.h          // TCP_NODELAY, SO_REUSEPORT, etc.
```

### 6. **Load Balancing & Worker Processes**

```

// Need: Multi-process architecturecore/├── lpd_master_process.c/.h         // Master process management├── lpd_worker_process.c/.h         // Worker process handling├── lpd_shared_memory.c/.h          // Inter-process communication├── lpd_load_balance.c/.h           // Request distribution
```

### 7. **Monitoring & Instrumentation**

```

// Need: Performance monitoringcore/├── lpd_stats.c/.h                 // Connection/request statistics  ├── lpd_metrics.c/.h               // Performance metrics├── lpd_health_check.c/.h          // System health monitoring
```

## Key Architectural Challenges for 100K

### **Memory Per Connection**

- Each connection needs ~2-8KB of memory
- 100K connections = 200MB-800MB just for connection state
- Need efficient memory pools and recycling

### **File Descriptor Limits**

```

// Need to handle OS limits#define LPD_MAX_CONNECTIONS 100000// Requires: ulimit -n 100000+, SO_REUSEPORT, etc.
```

### **Event Loop Efficiency**

- Single-threaded event loop becomes bottleneck
- Need worker processes (typically 1 per CPU core)
- Each worker handles ~10K-25K connections

### **TCP Stack Optimization**

```

// Need kernel bypass or optimizationint tcp_nodelay = 1;int so_reuseport = 1;int tcp_defer_accept = 1;// Plus: receive/send buffer tuning
```

## The "100K Problem" Reality

Even with all the static file components, you'd essentially have built **nginx without the 100K connection architecture**. The missing pieces are:

1. **Multi-process worker model** (fork workers, shared listening socket)
2. **Advanced event batching** (process many events per epoll_wait)
3. **Zero-allocation hot paths** (no malloc/free in request handling)
4. **Connection pooling & reuse** (minimize object creation/destruction)
5. **OS kernel optimizations** (TCP settings, file descriptor limits)

So with the nginx static components, you'd have **excellent static file performance** but would likely max out around **5K-10K concurrent connections** on typical hardware without the additional scalability architecture.