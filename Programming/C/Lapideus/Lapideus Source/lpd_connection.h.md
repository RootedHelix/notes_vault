Related to [[Lapideus]], [[Lapideus Source]]
#lpd_connection_h #lapideus_source 

#### `lpd_connection_tcp_nopush_e enum`
- This is the eumeration that tracks whether or not it has (or should have) the kernel "corking" option enabled on the socket. 
- - **`NGX_TCP_NOPUSH_UNSET`**
    
    - “We haven’t made a decision / haven’t called `setsockopt()` for corking yet.”
    - Typical initial state for TCP connections.
    - Lets Nginx lazily enable it only when it’s actually beneficial (e.g., when about to send headers + file).
- **`NGX_TCP_NOPUSH_SET`**
    
    - “Corking is currently enabled on this socket.”
    - Nginx has already done the OS call (like `TCP_CORK`/`TCP_NOPUSH`) and remembers that so it doesn’t spam `setsockopt()` on every write.
- **`NGX_TCP_NOPUSH_DISABLED`**
    
    - “Do not use corking on this connection.”
    - This is used when it’s **not applicable** (e.g., not a TCP socket such as a Unix domain socket) or otherwise intentionally suppressed. It prevents Nginx from even trying.

# Structs 
### lpd_connection_s 

- `struct sockaddr* sockaddr`  **and** `socklen_t socklen`  - holds the remote(client) enpoint information

-  **`local_sockaddr`**: A pointer to a socket address structure that contains the local IP address and port that the server/application is bound to for this particular connection. This represents "our side" of the connection.
    
- **`local_socklen`**: The size/length of the structure in bytes. This is needed because can represent different address families (IPv4, IPv6, Unix domain sockets) which have different sizes. `local_sockaddr``sockaddr`
- **`addr_ntop`**: It controls how nginx turns the listening socket address into text (ls->addr_text), specifically whether it should use nginx’s normal “network-to-presentation” formatting path (IPv4/IPv6 string formatting, optional port handling).
		Practical purpose:
			It tells nginx: “for this listening socket, format/store a canonical textual address.”
			That text is used in logs, diagnostics, and places where nginx needs a human-readable listen address.
			It’s mostly relevant for cases where nginx can’t just reuse a prebuilt literal string and instead should derive it from the sockaddr.
			So conceptually, addr_ntop is a formatting/control flag for address string generation, not a networking behavior toggle (it doesn’t change how accept/connect works, only how the address is represented).