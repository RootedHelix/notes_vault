Related to [[Lapideus]], [[Lapideus Source]]
#lpd_connection_c #lapideus_source 


# TODO notes



# AI Goals

For just building a minimal event-driven server, the absolute minimum from ngx_connection.c is:

1. Allocate/init connection: ngx_get_connection()
2. Close connection: ngx_close_connection()
3. Register/unregister events with event loop: ngx_add_conn() / ngx_del_conn()
4. Non-blocking I/O: ngx_event_recv() / ngx_event_send()

Everything else in nginx is enhancements, logging, or optimizations for production robustness.

If your goal is a minimal event-driven application server, the requirements change slightly compared to a raw TCP server. Let me break it down carefully.

1️⃣ What an Application Server Needs Beyond Minimal Connection Functions

  A minimal ngx_connection.c skeleton handles sockets and I/O. But an application server typically adds:

|                     |                                                                 |
| ------------------- | --------------------------------------------------------------- |
| Component           | Why it’s needed                                                 |
| Request parsing     | HTTP, gRPC, or custom protocol parsing after reading raw bytes. |
| Routing / dispatch  | Map requests to handlers (e.g., /api/foo → function).           |
| Response building   | Serialize and send responses.                                   |
| Timeouts            | Prevent slow clients from holding resources indefinitely.       |
| Concurrency control | Avoid resource starvation under high load.                      |
| Optional SSL/TLS    | Encrypt traffic.                                                |

So while ngx_get_connection(), ngx_close_connection(), ngx_event_recv(), ngx_event_send(), and ngx_add_conn() provide the skeleton, the application layer adds logic on top of the raw connection.


2️⃣ How Minimal Functions Fit an App Server

1. Accept connections → ngx_get_connection()
2. Add to event loop → ngx_add_conn()
3. When ready to read → ngx_event_recv() → pass data to application parser
4. Process request → application logic (HTTP parser, database query, computation)
5. Send response → ngx_event_send()
6. Close connection → ngx_close_connection() (or keep alive)

3️⃣ Challenges for High Concurrency

Even if the raw connection functions can handle 100k connections, an application server faces extra constraints:

|   |   |
|---|---|
|Constraint|Effect|
|Request parsing|CPU-bound; can become bottleneck if requests are heavy.|
|Response serialization|Large payloads consume buffers; slow clients can cause backpressure.|
|Memory usage|Each connection may need per-request buffers or structures.|
|SSL/TLS|Significantly increases CPU usage and memory per connection.|
|Slow clients|Without timeouts or backpressure, 100k connections can stall the server.|

So minimal connection functions are still sufficient to maintain the sockets, but your application logic determines whether the server can realistically serve 100k simultaneous requests.

4️⃣ Practical Considerations

- Use non-blocking I/O for application logic too (don’t do blocking DB queries or file reads on the main event loop).
- Consider worker processes (like nginx) to scale across CPU cores.
- Use buffer pooling to reduce per-connection memory usage.
- Add timeouts for idle clients or slow reads/writes.

✅ Summary

- Minimal connection functions are enough to maintain the skeleton of an event-driven application server.
- Handling real application logic safely at high concurrency requires:  

- Non-blocking request processing
- Timeouts
- Efficient memory/buffer management
- Possibly multiple worker processes
   
If you want, I can draw a diagram showing how the minimal connection functions integrate with application logic in a high-concurrency server, so you can see which parts handle sockets vs. which handle the application.

|                        |                                                        |
| ---------------------- | ------------------------------------------------------ |
| Function               | Minimal Implementation                                 |
| ngx_get_connection()   | Allocate struct, init fd, last_activity, write_pending |
| ngx_close_connection() | Close fd, free struct (or return to pool)              |
| ngx_add_conn()         | Register fd with OS event mechanism (epoll/kqueue)     |
| ngx_del_conn()         | Remove fd from OS event mechanism                      |
| ngx_event_recv()       | Non-blocking recv, handle EAGAIN/EINTR                 |
| ngx_event_send()       | Non-blocking send, handle partial writes               |
|                        |                                                        |
|                        |                                                        |
|                        |                                                        |


### lpd_close_connection(lpd_connection_t* connection)

- uses a polymorphic architecture to communicate to different system implementations through a generic interface called lpd_event_actions_t who's Linux implementation is in *lpd_epoll_module.c* . 
- Checks the state of the lpd_connection_t struct and makes sure the proper flags are switched such that the connection can be closed off through the respective mentioned back-end functions


