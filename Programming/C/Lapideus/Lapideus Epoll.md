Exactly right — that’s the key insight behind why epoll (and similar mechanisms like kqueue on BSD/macOS or io_uring) outperform older system calls like select() or poll() at scale.

  

Let’s break it down clearly:

  

  

  

  

🧠 The core difference: 

stateful vs stateless monitoring

  

  

  

🧩 Older APIs — 

select()

 and 

poll()

  

  

- Every time you call them, you pass in a full list of file descriptors (FDs) to check.
- The kernel must iterate over the entire list on each call to see which are ready.
- That means time complexity ≈ O(n) per call, where n = total number of FDs.
- For high concurrency (say 10,000 sockets), that becomes very expensive.

  

poll(fds, nfds, timeout);

^ Every call, it re-checks all nfds entries, even if 99% are idle.

  

  

  

  

⚡ 

epoll

 — event-based and stateful

  

  

- You register FDs once with the kernel (epoll_ctl()).
- The kernel keeps an internal watch list of which FDs to monitor.
- When something happens (read/write/error), it marks that FD as ready internally.
- You then just call:

  

int n = epoll_wait(epfd, events, maxevents, timeout);

  

- …and the kernel returns only the ready FDs, without scanning the whole list.

  

  

✅ Result: you only pay O(ready_fds) instead of O(total_fds) per loop.

  

  

  

  

📈 Example comparison

  

|                    |                                |                       |                          |
| ------------------ | ------------------------------ | --------------------- | ------------------------ |
| Mechanism          | Kernel memory of FDs?          | Per-loop cost         | Scalability              |
| select()           | ❌ No (you pass them each time) | O(n)                  | Poor                     |
| poll()             | ❌ No                           | O(n)                  | Better, but still linear |
| epoll              | ✅ Yes (registered once)        | O(k) (k = #ready FDs) | Excellent                |
| kqueue (BSD/macOS) | ✅ Yes                          | O(k)                  | Excellent                |

  

  

  

  

🧩 The three key syscalls in epoll

  

int epoll_create1(0);                   // Create an epoll instance

epoll_ctl(epfd, EPOLL_CTL_ADD, fd, ...); // Register an FD with events

epoll_wait(epfd, events, maxevents, timeout); // Wait for ready FDs

The kernel keeps track of readiness internally using efficient data structures (usually a red-black tree + ready list).

  

  

  

  

🕹️ Why it’s perfect for event-driven servers

  

  

Because in an event loop (like your web server’s core):

  

- You don’t waste CPU scanning inactive sockets.
- You get notified only when something actually happens.
- Combined with non-blocking I/O, you can handle tens of thousands of connections in a single thread.

  

  

That’s exactly how NGINX, Node.js, and your Lapideus server achieve high concurrency.

  

  

  

  

🧮 Visualization

  

select/poll:

  loop:

    kernel checks every fd → returns ready ones → repeat

  

epoll:

  register fds once

  loop:

    kernel only gives back ready fds → no scanning → repeat

  

  

  

✅ Summary

|   |   |   |
|---|---|---|
|Feature|select/poll|epoll|
|Re-register each call|Yes|No|
|Kernel stores state|No|Yes|
|Returns all fds|Yes|Only ready|
|Performance|O(n)|O(k)|
|Scales to 100k+ connections|❌|✅|

  

  

  

If you want, I can show you a short annotated snippet comparing a poll() loop vs an epoll() loop — side-by-side, to show where the big efficiency difference happens. Would you like that?

Not exactly — ngx_event_accept() itself is not inside the ngx_epoll_module.c file. Let me clarify where it lives and how it relates to the epoll module.

  

  

  

  

1️⃣ Role of 

ngx_event_accept()

  

  

- ngx_event_accept() is responsible for accepting new client connections on a listening socket.
- It’s generic: it doesn’t know or care whether the worker is using epoll, kqueue, or another event mechanism.
- Its job:  
    

1. Call accept() on the listening socket.
2. Allocate a ngx_connection_t.
3. Set the new socket non-blocking.
4. Add it to the event loop (via read->handler callback).

-   
    
- It’s typically defined in:

  

src/event/ngx_event_accept.c

or in some versions inside src/os/unix/ngx_connection.c depending on NGINX source layout.

  

  

  

  

2️⃣ How it interacts with 

ngx_epoll_module

  

  

- ngx_epoll_module.c contains the epoll-specific implementation of the event mechanism:  
    

- epoll_create()
- epoll_ctl()
- epoll_wait()
- Event registration functions (add_event, del_event, etc.)

-   
    
- When a listening socket is registered for read events in epoll:  
    

- NGINX sets c->read->handler = ngx_event_accept;
- The worker loop waits in epoll_wait().
- When epoll signals that the listening socket is readable, the worker calls:

-   
    

  

c->read->handler(c->read); // which is ngx_event_accept

So ngx_event_accept() is the handler, and ngx_epoll_module is the epoll engine that triggers the handler.


3️⃣ Visualization of the flow

  

Master/worker creates listening socket

         |

         v

Worker registers socket with epoll

  epoll_ctl(EPOLLIN)

         |

         v

epoll_wait() blocks until socket ready

         |

         v

epoll signals readable FD

         |

         v

c->read->handler(c->read) → ngx_event_accept()

         |

         v

accept() new connection → set non-blocking → add to event loop

  

- Key point: ngx_event_accept() is generic, epoll just triggers it when a connection is ready.
- If NGINX used kqueue or select, the handler would still be ngx_event_accept(), only the underlying mechanism changes.

  

  

  

  

  

✅ Summary

  

|   |   |
|---|---|
|Component|Role|
|ngx_event_accept()|Accepts new connections, sets non-blocking, adds them to event loop|
|ngx_epoll_module|Implements epoll-specific event engine: wait, add, delete, trigger callbacks|
|Connection readiness|Epoll tells the worker which sockets are ready, then ngx_event_accept() is called|

  

  

  

If you want, I can make a diagram showing exactly how ngx_event_accept() fits into the epoll event loop, with reading/writing on client sockets included. It makes the master → worker → epoll → accept → client flow very clear.

  

Do you want me to make that?