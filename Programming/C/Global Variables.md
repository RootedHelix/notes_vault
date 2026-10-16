Related to [[C]], [[Stack and Heap Structure in memory]], 
#C #stack_and_heap

**global and variables are allocated storage automatically`static`**, and in embedded C they usually live in RAM for the life of the program

- **`.bss`** if zero-initialized
- **`.data`** if initialized with non-zero values

The startup code handles this before your application begins.
hey **already have memory reserved**, and if they’re /global without an initializer, they are typically **automatically set to zero at boot**. `static`