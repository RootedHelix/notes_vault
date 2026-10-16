Related to [[C]], [[CMake]]
#C #cmake #cmake_general_syntax


## #add_executable

**Definition**

>`add_executable()` creates a **runnable program target** in CMake.

Example:

```CMake
add_executable(ai_json_validator    main.c    validator.c    file_reader.c)
```

This tells CMake:

 Compile these `.c` files and link them into an executable program named `ai_json_validator`.

So when CLion asks for the debug **target**, this is usually what you choose.

Example full setup:

```CMake
add_library(JsonLib STATIC    json_parser.c    json_validator.c)target_include_directories(JsonLib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
add_executable(ai_json_validator   main.c)
target_link_libraries(ai_json_validator PRIVATE JsonLib)
```

Meaning:

```
JsonLib  = reusable library 
codeai_json_validator  = actual runnable programai_json_validator links against JsonLib
```

Key distinction:

```
add_library(...)
```

creates reusable compiled code.

```
add_executable(...)
```

creates the program you actually run/debug.


## #add_library function

**Definition**

>Creates a **library target** in `CMake`. A library target is reusable compiled code that other targets can link against.

Example:

```CMake
add_library(DataStructuresLib STATIC    LinkedList.c    ring_buffer.c    json_form_builder.c)
```

This tells CMake:

Compile these `.c` files into a static library named `DataStructuresLib`.

### Qualifiers

```
PRIVATE
```

Applies only to the target itself.

```
PUBLIC
```

Applies to the target itself **and** anything that links against it.

```
INTERFACE
```

Applies only to anything that links against the target, not the target itself.

### Examples

Then an executable can use it:

```CMake
add_executable(my_program main.c)
target_link_libraries(my_program PRIVATE DataStructuresLib)
```

Meaning:

```CMake
main.c executable
        links against
DataStructuresLib        
	built from
LinkedList.c, ring_buffer.c, json_form_builder.c
```

## #target_include_directories

**Definition**

> `target_include_directories()` tells CMake where a target should look for header files.

Example:

```CMake
target_include_directories(DataStructuresLib PUBLIC    src/DataStructures)
```

The basic pattern is:
```CMake
target_include_directories(target_name PUBLIC include_directory)
```


This means code using `DataStructuresLib` can include headers from:

```
src/DataStructures
```

So this works:

```
#include "LinkedList.h"
#include "ring_buffer.h"
```

instead of needing:

```
#include "src/DataStructures/LinkedList.h"
```
### Qualifiers

The important visibility options are:

```
PRIVATE
```

Only this target needs the include path.

```
PUBLIC
```

This target needs it, and anything linking to this target also needs it.

```
INTERFACE
```

This target itself does not need it, but anything linking to it does.