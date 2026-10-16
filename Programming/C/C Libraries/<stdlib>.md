
## strtol

long strtol(const char *nptr, char **endptr, int base)


- These Pertain to the third parameter:
### What counts as a “valid digit”?

- Base 2: `0–1`
    
- Base 8: `0–7`
    
- Base 10: `0–9`
    
- Base 16: `0–9` + `a–f`/`A–F`
    
- … up to Base 36: `0–9` + `a–z`/`A–Z` (`a/A`=10 … `z/Z`=35)
    
- Case is **insensitive** for letters.
    

If a character isn’t in that set, parsing stops **before** it.

You’ve almost got it—just one tweak:

- **1st param (`nptr`)**: pointer to the **start** of the C string to parse.
    
- **3rd param (`base`)**: the **radix** (2–36 or 0). This **implies which ASCII chars are valid digits** (`0–9`, and for bases >10, `a–z`/`A–Z` = 10–35).
    
- **2nd param (`endptr`)**: **output** location where `strtol` writes a pointer to the **first character _after_ the parsed number**. It doesn’t “define” the end—you pass in a `char**`, and `strtol` fills it.
    

Tiny demo:

```c
char *s = "123abc";
char *end;
long v = strtol(s, &end, 10);
// v == 123
// end points at s+3 (the 'a'), i.e., *end == 'a'
```

If you want to ensure the whole string is a valid number (after trimming), check:

```c
end != s        // at least one digit parsed
&& *end == '\0' // nothing left over
```