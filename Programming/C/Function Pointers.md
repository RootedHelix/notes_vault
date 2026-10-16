Related to [[C]], [[Pointers]]
#C #function_pointers

Here is an example within [[hlx_event.h]]

`typedef void (*hlx_event_handler_t)(const hlx_event_t* event);`

This statement is used to point to multiple other  different functions that you want to use but they must maintain the same [[Function Signature]], except for the name
