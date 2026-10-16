## TLS vs SSL

Great question — the relationship between **SSL**, **TLS**, and the **server** is mostly historical, but it's crucial to understand the evolution and how they work together (or rather, how **TLS replaced SSL**).

---

## 🔍 First: SSL vs TLS

|Term|Meaning|
|---|---|
|**SSL (Secure Sockets Layer)**|The original protocol for secure communication over the internet (developed by Netscape in the 1990s).|
|**TLS (Transport Layer Security)**|The modern, secure replacement for SSL — standardized and maintained by the IETF.|

> Today, **TLS is used exclusively** — SSL is deprecated and insecure.

---

## 🧠 So Why Do People Still Say “SSL”?

- "SSL" is still used as a **colloquial term**:
    
    - You'll see it in names like “SSL certificate,” “SSL/TLS,” or “HTTPS uses SSL” — but in reality, **TLS is what's used**.
        
    - Even OpenSSL (the library) supports only TLS now — SSLv2 and SSLv3 are fully disabled.
        

---

## 🔧 How TLS/SSL Interacts with a Server

Here’s how it plays out in a real server (e.g., your C web server, NGINX, Apache, etc.):

### 1. 📞 **TCP Connection**

- The client (e.g., browser) connects to the server over TCP (`port 443` for HTTPS).
    

### 2. 🤝 **TLS Handshake Begins**

- The server and client negotiate a **TLS session** (not SSL).
    
- This includes:
    
    - Server sends its certificate.
        
    - Key exchange (e.g., ECDHE).
        
    - Session key generation.
        

### 3. 🔐 **Secure Channel Established**

- Once the handshake is complete, the server and client:
    
    - Encrypt all data using the session key.
        
    - Use symmetric encryption (e.g., AES-GCM).
        
    - Ensure data integrity.
        

### 4. 🌐 **Application Data Sent**

- For HTTPS:
    
    - The server reads encrypted HTTP requests (via `SSL_read()`)
        
    - It sends back encrypted responses (via `SSL_write()`)
        

---

## ⚙️ Libraries That Handle This

|Library|Role|
|---|---|
|**OpenSSL / LibreSSL / BoringSSL**|TLS/SSL implementation used by many servers|
|**GnuTLS**, **mbedTLS**|Alternatives, sometimes used in embedded systems or Linux software|
|**Your Server Code**|Calls library APIs like `SSL_accept()`, `SSL_read()`, `SSL_write()` to manage secure connections|

---

## 💡 Summary

- **SSL is the old protocol**; **TLS is what’s actually used today**.
    
- Your **server uses a TLS library (like OpenSSL)** to negotiate and maintain encrypted connections.
    
- “SSL certificates” are **actually X.509 certificates used by TLS** — the name is just legacy.
    

---

Let me know if you'd like:

- A diagram of OpenSSL integration in a C server
    
- How NGINX wraps TLS over sockets
    
- Or a version-by-version comparison (SSLv3 vs TLS 1.0–1.3)