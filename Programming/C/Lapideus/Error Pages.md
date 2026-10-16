
Here are the definitions of those HTTP error status codes:

### **400 – Bad Request**

This means the server could not understand the request due to invalid syntax.  
Examples:

- Malformed HTTP request line (missing method/URI/version).
    
- Invalid or improperly encoded headers.
    
- An oversized request line or header field (if not explicitly covered by 414/431).
    

The client should fix the request and try again.

---

### **404 – Not Found**

This means the server cannot find the resource that the client requested.  
Examples:

- A requested URL does not map to any file, route, or handler on the server.
    
- The resource was deleted or moved without redirection.
    

It does not reveal whether the condition is temporary or permanent (though **410 Gone** is used when it’s permanent).

---

### **431 – Request Header Fields Too Large**

This means the server is refusing to process the request because one or more of the HTTP headers are too large.  
Examples:

- Cookie headers with excessive size.
    
- Requests with an unusually large number of custom headers.
    

The client should reduce the size of the request headers and try again (e.g., clear cookies, compress metadata).

---

 For your application server, **400** is your “generic bad input” page, **404** is the “resource missing” page, and **431** is more niche but important when handling overly large request headers (especially in systems with lots of cookies or auth tokens).