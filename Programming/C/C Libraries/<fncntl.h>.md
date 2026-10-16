Great question! The `fcntl.h` library (header file) is a standard part of the **POSIX/UNIX C library**. It provides **constants, macros, and function prototypes** for:

- **File control operations**
    
- **File status flags**
    
- **File descriptor manipulation**
    
- **Record locking**
    
- **Other advanced file operations**
    

---

## **Key Points about `fcntl.h`**

- **Header file:**
    
    ```c
    #include <fcntl.h>
    ```
    
- **Used for:**
    
    - Opening files (`open`, `creat`) with specific flags (like `O_RDONLY`, `O_CREAT`, `O_APPEND`)
        
    - Setting and getting file descriptor flags (e.g., non-blocking mode)
        
    - File and record locking (`fcntl` function)
        
    - Duplicating file descriptors
        

---

## **Most Common Elements Provided by `fcntl.h`**

- **File open flags:**
    
    - `O_RDONLY`, `O_WRONLY`, `O_RDWR`, `O_CREAT`, `O_TRUNC`, `O_APPEND`, etc.
        
- **File descriptor flags:**
    
    - `FD_CLOEXEC`, `F_GETFL`, `F_SETFL`, etc.
        
- **Function prototypes:**
    
    - `int fcntl(int fd, int cmd, ...);`
        
    - `int open(const char *pathname, int flags, ...);`
        
    - `int creat(const char *pathname, mode_t mode);`
        

---

## **Example Use**

```c
#include <fcntl.h>
#include <unistd.h>

int fd = open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
if (fd != -1) {
    // File is open for writing, created if not exists, truncated if exists
    // ... do file operations ...
    close(fd);
}
```

---

## **Why do you need it?**

- If you want to use **low-level POSIX file I/O**, especially with custom flags or advanced file operations, you need to include `fcntl.h`.
    
- It’s essential for writing cross-platform, UNIX-style code—like NGINX does.
    

---

## **TL;DR:**

> **`fcntl.h` is a standard POSIX C header that provides file control and descriptor manipulation functions and macros, essential for low-level file operations on UNIX systems.**

Let me know if you want examples of specific `fcntl` usage or details on cross-platform file I/O!


## Fcntl()

fcntl() is the generic tool to inspect or change properties of any file descriptor. When you use it on a socket:

  

- With F_GETFL → you read the socket’s current status flags (e.g. is it non-blocking, append mode, etc.).
- With F_SETFL → you write new flags to the socket (e.g. turn on O_NONBLOCK).


So you’re not changing the underlying TCP socket itself — you’re telling the OS: “treat I/O on this file descriptor with these rules.