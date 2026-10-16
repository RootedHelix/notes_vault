Perfect — let’s zoom in on the 10–15 design patterns that are actually useful for a C web server project (like the one you’re building). These come from the GoF catalog, plus some concurrency/networking patterns that Nginx and similar servers use.


🔑 Core Patterns for a C Web Server


Creational

1. Factory Method – Create objects (like ngx_connection_t) in a consistent way, often from a pool.
2. Singleton – Ensure only one instance of global state (cycle, logging system, config).
3. Object Pool – Manage reusable connections/buffers instead of malloc/free every Perfect — let’s zoom in on the 10–15 design patterns that are actually useful for a C web server project (like the one you’re building). These come from the GoF catalog, plus some concurrency/networking patterns that Nginx and similar servers use.

🔑 Core Patterns for a C Web Server
L
  

Creational

1. Factory Method – Create objects (like ngx_connection_t) in a consistent way, often from a pool.
2. Singleton – Ensure only one instance of global state (cycle, logging system, config).
3. Object Pool – Manage reusable connections/buffers instead of malloc/free every time.


Structural

4. Adapter – Wrap system calls or OS APIs in your own interface (e.g., wrap epoll in event.c).
5. Facade – Provide a simplified API to complex subsystems (e.g., cycle provides a unified way to start/stop server).
6. Composite – Useful if you implement config trees (hierarchical config blocks like in nginx.conf).


Behavioral


7. Strategy – Allow different algorithms for handling events (HTTP handler, static file handler, proxy handler).
8. Observer – Notify multiple subsystems when an event happens (connection closed → free resources, log it, update stats).
9. State – Represent connection states (LISTENING, READING, WRITING, CLOSING).
10. Command – Encapsulate tasks like “send response” or “close connection” as discrete commands that can be queued/executed.
11. Template Method – Define an algorithm skeleton (like event loop) and allow substeps to be overridden.

  

Concurrency / Networking (POSA Patterns)


      (Nginx)
12. Reactor – The core of your event.c: demultiplex I/O events and dispatch to handlers.
13. Proactor – Alternative model if you used async I/O (less common in plain C on Linux, but worth knowing).
14. Half-Sync/Half-Async – Divide synchronous app logic (e.g. parsing HTTP) from async event handling.
15. Thread Pool – If you decide to offload heavy tasks (like compression or database queries).

  

  

  

  

✅ With just these 15, you’ve covered the majority of the design space needed for a high-performance server. The rest of the “hundreds” of patterns are often irrelevant to systems-level C.

  

  

  

👉 Want me to make you a visual map (like a diagram) showing how these patterns interconnect in a web server? That way you’ll see at a glance which layer uses which pattern.
  

Structural

  

  

4. Adapter – Wrap system calls or OS APIs in your own interface (e.g., wrap epoll in event.c).
5. Facade – Provide a simplified API to complex subsystems (e.g., cycle provides a unified way to start/stop server).
6. Composite – Useful if you implement config trees (hierarchical config blocks like in nginx.conf).

  

  

  

  

  

Behavioral

  

  

7. Strategy – Allow different algorithms for handling events (HTTP handler, static file handler, proxy handler).
8. Observer – Notify multiple subsystems when an event happens (connection closed → free resources, log it, update stats).
9. State – Represent connection states (LISTENING, READING, WRITING, CLOSING).
10. Command – Encapsulate tasks like “send response” or “close connection” as discrete commands that can be queued/executed.
11. Template Method – Define an algorithm skeleton (like event loop) and allow substeps to be overridden.

  

  

  

  

  

Concurrency / Networking (POSA Patterns)

  

  

12. Reactor – The core of your event.c: demultiplex I/O events and dispatch to handlers.
13. Proactor – Alternative model if you used async I/O (less common in plain C on Linux, but worth knowing).
14. Half-Sync/Half-Async – Divide synchronous app logic (e.g. parsing HTTP) from async event handling.
15. Thread Pool – If you decide to offload heavy tasks (like compression or database queries).

  

  

  

  

✅ With just these 15, you’ve covered the majority of the design space needed for a high-performance server. The rest of the “hundreds” of patterns are often irrelevant to systems-level C.

  

  

  

👉 Want me to make you a visual map (like a diagram) showing how these patterns interconnect in a web server? That way you’ll see at a glance which layer uses which pattern.