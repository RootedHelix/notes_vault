Related to [[Lapideus Build Log]]
#nagles_algorithm

## What Is Nagle’s Algorithm?

**Nagle's algorithm** is a TCP optimization designed to **reduce network congestion caused by lots of tiny packets**.

It was introduced by **John Nagle** in 1984.

---

# The Problem It Solves

TCP allows you to send very small chunks of data.

Example:

```c
send(sock, "H", 1);
send(sock, "i", 1);
```

Without optimization, that could produce **two separate TCP packets**.

Each packet has:

- TCP header (~20 bytes)
- IP header (~20 bytes)
- Ethernet framing
- Plus your tiny payload

So you might send:

> 40+ bytes of overhead for 1 byte of data.

That’s inefficient.

---

# What Nagle Does

Nagle’s algorithm says:

> “If there is unacknowledged data in flight, don’t send small packets.  
> Buffer them until you can send a full-sized segment.”

More formally:

1. Send the first small packet immediately.
    
2. If there’s unacknowledged data outstanding:
    
    - Buffer new small writes
    - Wait for ACK
    - Or wait until you can fill a full MSS-sized packet

---

# Result

Fewer small packets  
Less congestion  
Better throughput on slow networks

---

# Where It Hurts

Nagle improves throughput  
But can increase latency.

Especially for:

- Interactive apps
- Games
- SSH
- Real-time chat typing
- RPC systems

Because it may wait for an ACK before sending the next small chunk.

---

# ⚠️ The Classic Problem: Nagle + Delayed ACK

TCP also uses **Delayed ACK**:

The receiver may wait ~40–200ms before sending an ACK.

Now combine:

1. Sender waits for ACK (Nagle)
    
2. Receiver waits before sending ACK (Delayed ACK)
    

Result:  
👉 200ms random delays

This is called the **Nagle/Delayed ACK interaction problem**.

---

# How To Disable It

You disable Nagle using:

```c
int flag = 1;
setsockopt(sock, IPPROTO_TCP, TCP_NODELAY, &flag, sizeof(flag));
```

This enables **TCP_NODELAY**.

Used in:

- Games
    
- Low-latency trading systems
    
- Some HTTP servers
    
- Real-time systems
    

---

# In Your Server Context

Since you're building an event-driven server:

If you're:

- Sending HTTP headers + body in small writes
- Streaming responses
- Building interactive protocols

You might want to:

- Either buffer properly in userspace
- Or enable TCP_NODELAY

NGINX, for example, carefully manages this behavior.

---

# 🎯 Simple Mental Model

Nagle:

> “Don’t send tiny packets if you already have one in flight.”

It optimizes bandwidth efficiency  
At the potential cost of latency.

---

If you want, I can explain:

- 📡 How this interacts with epoll
    
- 🧠 How modern congestion control (like CUBIC) relates
    
- 🔬 How to observe Nagle in Wireshark