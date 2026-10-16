
Here’s the clean, systems-level distinction between **Unix domain sockets** and **network (IP) sockets**, framed the way you’d care about it when writing servers or studying NGINX-style architecture.

---

## High-level idea

- **Unix domain socket** → _local IPC_ (same machine only)
    
- **Network socket** → _network IPC_ (local or remote, over IP)
    

They use the **same socket API**, but differ in **addressing, transport path, performance, and security model**.

---

## Addressing & namespace

### Unix domain socket

- Addressed by a **filesystem path** (or abstract namespace on Linux)
    
- Example:
    
    ```c
    struct sockaddr_un addr;
    addr.sun_family = AF_UNIX;
    strcpy(addr.sun_path, "/run/myapp.sock");
    ```
    
- No IP, no port
    
- Exists as a special file (`srwxr-xr-x`)
    

![Image](https://inside.java/images/blog/jep380.jpg)

![Image](https://www.c-pointers.com/_images/plantuml-4881a39789ecae0c0ffa2f55f83760c3bdb420c2.png)

![Image](https://media.licdn.com/dms/image/v2/D4E22AQGtS3ETVdeaTQ/feedshare-shrink_800/B4EZbNej8YHMAg-/0/1747204065844?e=2147483647&t=Uwp0uy1iskcaUagUqeWoqbApSnFsHm7dyBtn4opB3mM&v=beta)

---

### Network socket (TCP/UDP)

- Addressed by **IP address + port**
    
- Example:
    
    ```c
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;
    ```
    
- Uses the network stack (IPv4 / IPv6)
    

![Image](https://www.researchgate.net/publication/349147426/figure/fig12/AS%3A989602185744386%401612950955194/Simplified-diagram-of-TCP-IP-based-socket-communication-Adapted-from-18.ppm)

![Image](https://www.eg.bucknell.edu/~cs363/2016-spring/labs/tcplab-236x300.png)

![Image](https://notes.shichao.io/unp/figure_3.7.png)

---

## Transport path (this is the _big_ difference)

### Unix socket path

```
process → kernel → process
```

- No routing
    
- No NIC
    
- No IP headers
    
- No checksum offloading
    
- No congestion control
    

### Network socket path

```
process → kernel → TCP/IP stack → NIC → network → NIC → TCP/IP stack → kernel → process
```

This is why Unix sockets are **faster and lower latency**.

---

## Performance characteristics

|Aspect|Unix Socket|Network Socket|
|---|---|---|
|Latency|**Very low**|Higher|
|Throughput|**Higher (local)**|Lower|
|CPU cost|**Lower**|Higher|
|Copies|Fewer|More|
|Zero-copy|Easier|Harder|

In practice:

- Unix sockets often outperform loopback TCP (`127.0.0.1`)
    
- NGINX defaults to **Unix sockets for local upstreams** (e.g., Gunicorn)
    

---

## Security model

### Unix domain sockets

- Access controlled by **filesystem permissions**
    
- Can enforce:
    
    ```bash
    srw-rw---- nginx www-data /run/app.sock
    ```
    
- Supports **credential passing** (`SO_PEERCRED`)
    
    - Server can get UID/GID/PID of client
        
- No exposure to the network
    

### Network sockets

- Access controlled by:
    
    - Firewall rules
        
    - TLS
        
    - Authentication layers
        
- Potentially reachable by _anyone_ if misconfigured
    

---

## Feature differences

|Feature|Unix Socket|Network Socket|
|---|---|---|
|Remote access|❌ No|✅ Yes|
|Filesystem permissions|✅ Yes|❌ No|
|Credential passing|✅ Yes|❌ No|
|TLS|❌ Not needed|✅ Often required|
|NAT / routing|❌ No|✅ Yes|

---

## Typical use cases

### Use **Unix sockets** when:

- Client and server are on the **same machine**
    
- You want **maximum performance**
    
- You want **simple security**
    
- Example:
    
    - NGINX → Gunicorn
        
    - Supervisor → worker processes
        
    - Local databases (Postgres socket)
        

### Use **network sockets** when:

- Clients may be **remote**
    
- You need **horizontal scaling**
    
- You need **standard protocols**
    
- Example:
    
    - Browsers → web servers
        
    - Microservices across hosts
        
    - Public APIs
        

---

## Why NGINX (and you) care

In an NGINX-style architecture (which you’re actively mirroring):

- **Listener socket**: network socket (TCP)
    
- **Upstream socket**: often Unix domain socket
    
- Reduces latency and syscall overhead between tiers
    
- Avoids TLS internally
    

That’s exactly why NGINX recommends:

```nginx
upstream app {
    server unix:/run/gunicorn.sock;
}
```

---

## Mental model (one sentence)

> **Unix sockets are shared-kernel pipes with file permissions; network sockets are routed packets with IP semantics.**

If you want, next we can:

- Compare **Unix socket vs loopback TCP benchmarks**
    
- Walk through **accept/connect internals**
    
- Discuss **how epoll treats both identically**
    
- Map this directly to your `lpd_connection_t` design