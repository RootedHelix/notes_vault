
# Samuel O’Hearne

Phoenix, AZ • samo12@me.com • 480-799-6604 
- - -

### SUMMARY

Early‑career systems engineer focused on C on Linux. Built a high‑performance HTTP application server modeled after NGINX with event‑driven concurrency and low‑level memory management. Comfortable reading upstream source, profiling, and shipping production‑style code. Seeking roles in C development, platform, or performance engineering.

### CORE SKILLS

C (C17), POSIX sockets, epoll/kqueue model, Linux internals, memory pools/allocators, concurrency & synchronization, zero‑copy I/O (sendfile), TCP/IP, HTTP/1.1 parsing, build systems (CMake), unit testing, valgrind, Git/GitHub, CI basics, logging/metrics.

### SELECT PROJECTS

**Lapideus — Lean NGINX‑style HTTP Server (C, Linux) • 2025**

- Architected an NGINX‑like event‑driven server targeting Linux; reused permissible NGINX components under license and implemented original modules around them.
    
- Implemented FSM‑based HTTP/1.1 request parser storing byte‑accurate pointers into buffers; supports keep‑alive and parser error states.
    
- Designed memory‑pool allocator to minimize heap fragmentation and speed per‑request allocations; added safe string utilities.
    
- Built epoll‑based event loop with reusable connection structs, posted‑event queues, and timers; prepared for multi‑worker design with a cycle‑state structure.
    
- Added logging pipeline with log levels and timestamp formatting; integrated debug macros.
    
- Status: **~85% complete**; next scope: TLS via OpenSSL/mbedTLS, request observability, worker sharding.
    

**Future Module — WAF extension for Lapideus (design in progress)**

- Designing a minimal WAF module with rate‑limit buckets per IP, request anomaly scoring, and pluggable rules; plan to contribute upstream for community feedback once stable.
    
### EXPERIENCE

**Cybersecurity Intern —  InfoArmor, Scottsdale, AZ • Summer 2019**

- Assisted with vuln triage and documentation for internal reports; ran basic network scans and packet captures under supervision (Nmap, Wireshark).
    
- Drafted SOP updates for artifact handling and ticket handoffs.
    

**Other Work (Food Service & Retail) — Various Employers • 2016–2024**

- Roles emphasizing reliability, shift leadership, and customer communication; promoted twice; handled cash and reconciliations.
    

### EDUCATION

**University coursework in Software Engineering** — 2 years completed • ASU, Phoenix

- Relevant courses: Data Structures & Algorithms, Operating Systems, Computer Systems (C), Secure Coding.
    

### SELECTED HIGHLIGHTS

- Reverse‑read large portions of upstream NGINX to mirror naming/architecture patterns appropriately in Lapideus.
    
- Analyzed security risks throughout the development of the project in order to achieve production level design