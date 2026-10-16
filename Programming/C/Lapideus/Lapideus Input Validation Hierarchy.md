
| Directory           | Typical Validation Policy                                 |
| ------------------- | --------------------------------------------------------- |
| src/os/             | Avoids input validation; assumes trusted calls            |
| src/core/           | Avoids input validation; internal/core helpers            |
| src/DataStructures/ | Avoids input validation; internal data structures         |
| src/http/           | Performs input validation; parses/validates raw input     |
| src/event/          | Performs input validation for boundaries (network events) |

 src/os, src/core, and src/DataStructures would match NGINX's pattern of "avoiding" redundant input validation internally, while src/http and src/event act as "boundaries" and should validate inputs.
