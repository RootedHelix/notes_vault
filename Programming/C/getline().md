Related to [[C]], [[<string.h>]]


## Personal Notes

> Function Signature: `ssize_t getline(char **lineptr, size_t *n, FILE *stream);`

char **lineptr      — pointer to your char* buffer. getline will malloc/realloc it as needed. Must                                     be  initialized to NULL if you want getline to allocate it fresh.
size_t *  n           — pointer to the buffer's current capacity. Set to 0 when starting fresh.
FILE *stream       — input source, e.g. stdin.
Returns ssize_t — number of characters read (including '\n'), or -1 on EOF/error.

Reads in the next line into

### Key features 

- If you are reading into a Stack Buffer but you 
## Ai Response: 
