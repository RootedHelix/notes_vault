
Low Address
+-------------------+ 
|    Code Segment   | ← Program instructions
+-------------------+
|    Data Segment   | ← Global/Static variables
+-------------------+
|     Heap (↑)      | ← Grows upward from here
|                   |
|    Free Space     | ← Buffer zone
|                   |
|     Stack (↓)     | ← Grows downward from here
+-------------------+
High Address

- The heap starts allocating from memory addresses right after the data segment and continues to allocate higher and higher memory addresses until the heap is full. If more memory is necessary for the heap it will use memory in the free space if available.
- The stack starts allocating at the highest available memory address and allocates addresses downwards. Similar to the heap, if more memory is needed for the stack, memory will be allocated from the free space if available. 

┌─────────────────────────┐
│        Stack            │ ← Local variables, function calls
├─────────────────────────┤
│        Heap             │ ← malloc(), dynamic allocation
├─────────────────────────┤
│        BSS              │ ← Uninitialized global/static variables
├─────────────────────────┤
│    Data Segment         │ ← Initialized global/static variables
├─────────────────────────┤
│    Text/Code Segment    │ ← Program instructions
└─────────────────────────┘