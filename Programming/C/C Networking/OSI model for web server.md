
[7] Application  ←  HTTP parser, user handlers
[6] Presentation <--- **Handling SSL/TLS encryption/decryption**
- (In NGINX, this is via the OpenSSL integration)
[5] Session      ←  lpd_connection_t, connection pool
[4] Transport    ←  OS sockets, epoll, select
[2–3] Data Link/Network   ←  Provided by OS/kernel, not touched by user code
[1] Physical      ←  Hardware, not seen by your program