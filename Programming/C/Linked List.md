Related to [[C]], [[Data Structures]]
#C #data_structures

## Personal Notes 

Linked list 
```C
struct node_s {
int data;
node_s* next;
}

struct linked_list_s {
node_s* head;
unsigned int length;
unsigned int max;
}

```

See `CLearning/DataStructures/data_practice.c`



# AI Response

Why do we initialize the node->next pointer to NULL?
- **To indicate the end of the list**
    
    - In a singly linked list, the last node's `next` pointer should always be `NULL` to signify the end of the list.
        
    - Without this, the pointer may contain **garbage memory** or an unexpected value, which could lead to:
        
        - Infinite loops
            
        - Undefined behavior
            
        - Memory access errors
            
- **Consistency in initialization**
    
    - When you create a **new standalone node**, you want it to be a **self-contained** unit.
        
    - Even if you haven't linked it to the list yet, initializing `next` to `NULL` ensures the node behaves properly as a tail node by default.
        
    - This makes it safer and easier to append it to the list later.


## **When to Use What?**

| Feature              | **Linked List**                     | **Dynamically Allocated Array**         |
| -------------------- | ----------------------------------- | --------------------------------------- |
| Insertions/Deletions | **Fast (`O(1)`)**                   | Slow (`O(n)`)                           |
| Random Access        | **Slow (`O(n)`)**                   | **Fast (`O(1)`)**                       |
| Memory Allocation    | **Flexible, grows dynamically**     | **Fixed size or requires reallocation** |
| Cache Performance    | Poor (pointers scattered in memory) | **Great (contiguous memory)**           |
| Memory Overhead      | **High (extra `next` pointers)**    | Low (only data stored)                  |

- **Use a linked list** when **frequent insertions/deletions** are required and you don’t need fast random access.
    
- **Use an array** when **fast indexing and better memory/cache performance** are needed.

