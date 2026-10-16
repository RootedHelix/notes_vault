
# Samuel O’Hearne

Phoenix, AZ • [email] • [phone] • [GitHub] • [LinkedIn]

---

## Variant A — Software Engineer (C / Systems)

### SUMMARY

Early‑career systems engineer focused on C on Linux. Built a high‑performance HTTP application server modeled after NGINX with event‑driven concurrency and low‑level memory management. Comfortable reading upstream source, profiling, and shipping production‑style code. Seeking roles in C development, platform, or performance engineering.

### CORE SKILLS

C (C17), POSIX sockets, epoll/kqueue model, Linux internals, memory pools/allocators, concurrency & synchronization, zero‑copy I/O (sendfile), TCP/IP, HTTP/1.1 parsing, build systems (CMake), unit testing, gdb/valgrind, perf/strace, Git/GitHub, CI basics, logging/metrics.

### SELECT PROJECTS

**Lapideus — Lean NGINX‑style HTTP Server (C, Linux) • 2025**

- Architected an NGINX‑like event‑driven server targeting Linux; reused permissible NGINX components under license and implemented original modules around them.
    
- Implemented FSM‑based HTTP/1.1 request parser (request line + header parsing) storing byte‑accurate pointers into buffers; supports keep‑alive and parser error states.
    
- Designed memory‑pool allocator (small/large blocks) to minimize heap fragmentation and speed per‑request allocations; added safe string utilities (length‑tracked).
    
- Built epoll‑based event loop with reusable connection structs, posted‑event queues, and timers; prepared for multi‑worker design with a cycle‑state structure.
    
- Added sendfile‑based static file serving path and structured logging pipeline with log levels and timestamp formatting; integrated debug macros.
    
- Results (on dev box): served static files to **N** concurrent connections with median latency of **X ms** and throughput of **Y MB/s**; CPU profile shows **~Z%** in syscalls. _(replace N/X/Y/Z with your benchmark numbers)_
    
- Status: **~85% complete**; next scope: TLS via OpenSSL/mbedTLS, request observability, worker sharding.
    

**Future Module — WAF extension for Lapideus (design in progress)**

- Designing a minimal WAF module with rate‑limit buckets per IP, request anomaly scoring, and pluggable rules; plan to contribute upstream for community feedback once stable.
    

### EXPERIENCE

**Cybersecurity Intern — [Firm Name], City, ST • Summer 2019**

- Assisted with vuln triage and documentation for internal reports; ran basic network scans and packet captures under supervision (Nmap, Wireshark).
    
- Drafted SOP updates for artifact handling and ticket handoffs.
    

**Other Work (Food Service & Retail) — Various Employers • 2016–2024**

- Roles emphasizing reliability, shift leadership, and customer communication; promoted twice; handled cash and reconciliations.
    

### EDUCATION

**University coursework in Software Engineering** — 2 years completed • [University], [City]

- Relevant courses: Data Structures & Algorithms, Operating Systems, Networks, Computer Systems (C), Secure Coding.
    

### SELECTED HIGHLIGHTS

- Reverse‑read large portions of upstream NGINX to mirror naming/architecture patterns appropriately in Lapideus.
    
- Built comprehensive unit tests for parser and allocator components; introduced debug builds with sanitizer flags where applicable.
    

---

## Variant B — Cybersecurity Analyst (Blue‑Team‑leaning)

### SUMMARY

Security‑minded C developer pivoting into blue‑team analysis and defensive engineering. Hands‑on with Linux networking and HTTP internals; building my own NGINX‑style server has sharpened protocol‑level troubleshooting and log analysis skills. Looking for SOC/Detection, Security Engineering, or AppSec‑adjacent roles.

### CORE SKILLS

Network fundamentals (TCP/IP, HTTP), Linux CLI, Wireshark/tcpdump, log analysis, basic SIEM concepts, scripting (Bash), C static analysis basics, secure coding patterns, rate‑limiting and request fingerprinting design, Git/GitHub, documentation.

### SECURITY PROJECTS

**Lapideus — Observability & Abuse‑mitigation features (C, Linux)**

- Implemented structured request logging with request/response metadata for downstream analysis; added severity levels and timestamps for correlation.
    
- Designed IP bucket rate‑limiting and anomaly score model (spec): per‑IP token bucket + path‑based heuristics aimed at slow‑loris style and burst behaviors.
    
- Built parser error paths that avoid data leakage and ensure deterministic 4xx/5xx responses to reduce the risk of memory disclosure.
    
- Future: WAF rules engine module with minimal false‑positive guardrails and transparent mode for learning.
    

**Security Lab Notes (self‑study)**

- Packet analysis exercises with pcaps of HTTP traffic; wrote short playbooks: “Identify half‑open connections,” “Spot header anomalies,” and “Map 4xx/5xx spikes to IP ranges.”
    
- Wrote hardening checklist for dev hosts: least‑privilege users, firewall rules, log rotation, and update cadence.
    

### EXPERIENCE

**Cybersecurity Intern — [Firm Name], City, ST • Summer 2019**

- Shadowed analysts in triage, reproduced PoCs in lab VMs, and wrote summaries for tickets; contributed notes used in weekly standups.
    

**Other Work (Food Service & Retail) — Various Employers • 2016–2024**

- Customer‑facing roles; trained new hires; maintained shift logs and incident notes.
    

### EDUCATION

**University coursework in Software Engineering** — 2 years completed • [University], [City]

- Relevant courses: Secure Coding, Operating Systems, Networks, Discrete Math.
    

### CERTS & NEXT STEPS (optional to list)

- Target: CompTIA Security+ (or Linux+), eJPT/eJPTv2 later; share ETA or in‑progress only when scheduled.
    

---

## One‑Page Formatting Notes (for both variants)

- Keep it to **one page**. Use 10–11pt body font, consistent bullets, and plenty of white space.
    
- Put **Projects** above Experience to foreground Lapideus.
    
- Use **numbers** where possible (req/s, latency, connections, LoC tested, coverage %). Bench on your box and record settings.
    
- Avoid future promises; mark in‑progress items clearly.
    
- ATS‑friendly: avoid text boxes and columns; export to PDF with selectable text.
    
- File name: `Samuel-OHearn_C-Systems_Resume.pdf` or `..._Cybersecurity.pdf`.




## Strategy to get pas ATS: 

Short answer: it’s close, but I’d make a few tweaks so it cruises through most ATS filters and surfaces to a human.

Here’s the punch-list I recommend (fast to do):

### 1) Put exact job titles at the top

Use **“Software Engineer (C / Linux Systems)”** on the C version and **“Cybersecurity Analyst (Blue Team / Detection)”** on the security version. ATS often scores exact title matches.

### 2) Mirror the keywords from target JDs

Add a “Skills” and a “Tools” line that includes the exact phrases employers use. Keep them comma-separated (no tables). Examples to slot in:

**C / Systems keyword bank**

- C (C17), Linux, POSIX, TCP/IP, UDP, HTTP/1.1, epoll, non-blocking I/O, concurrency, multithreading, memory allocator, zero-copy, sendfile, sockets, OpenSSL, TLS, CMake, gdb, valgrind, perf, strace, unit testing, CI/CD, Git, GitHub Actions, containers/Docker (if applicable).
    

**Cybersecurity keyword bank**

- SOC, SIEM (Splunk/Elastic if you’ve used), log analysis, incident triage, threat detection, IDS/IPS, packet analysis, tcpdump, Wireshark, Nmap, vulnerability assessment, Linux hardening, incident response, MITRE ATT&CK, regex/rules, bash scripting, WAF, rate limiting, anomaly detection.
    

(Only list tools you can actually use; ATS likes proximity matching, but don’t inflate.)

### 3) Quantify the project impact

ATS isn’t “reading,” but once a human opens it the numbers sell you. Add 2–3 hard metrics to Lapideus:

- “Served **N** concurrent connections; **X ms** p50 latency; **Y MB/s** static throughput.”
    
- “Parser covers **100%** of request-line states; **Z** unit tests; sanitizer builds passing.”
    
- “Reduced per-request allocs by **Q%** via memory pool.”
    

### 4) Add a brief one-line “Tech stack” on the Lapideus project

Right under the project name: “**Tech:** C, Linux, epoll, POSIX sockets, CMake, gdb/valgrind, sendfile, (OpenSSL planned).”

### 5) Move Projects above Experience (already done) and keep to one page

ATS is fine with that. Keep body 10–11pt, plain bullets, no columns, no text boxes.

### 6) Keep contact info in the body, not in headers/footers

Some parsers skip header/footer content. Make sure your name + links are in the normal body.

### 7) Use standard section names

“Summary,” “Skills,” “Projects,” “Experience,” “Education.” Avoid creative headings—ATS maps these labels.

### 8) Export to text-selectable PDF

No scanned PDFs. Test by trying to copy/paste text afterward.

### 9) Small wording tweaks for ATS matches

- Replace “NGINX-style” with “**NGINX** (open-source web server)–style architecture; re-used upstream components under license.”
    
- Include both “**Linux**” and “**GNU/Linux**” once (some JDs key on either).
    
- For security, include “**WAF** (web application firewall)” and “**detection engineering**” if the JD mentions them.
    

### 10) Education line

Spell out “**B.S. in Software Engineering (in progress / 2 years completed)**” with relevant courses listed as keywords: Operating Systems (C), Computer Networks, Secure Coding.

---

## Quick ATS checklist (yes/no)

- One page, .pdf, text selectable — **Yes**
    
- Standard headings & simple layout — **Yes**
    
- Exact job title mirrored — **Add**
    
- Keyword density aligned to JD — **Add a few lines**
    
- Numbers in project bullets — **Add**
    
- Contact info in body (not header/footer) — **Confirm**
    
- File name like `Samuel-OHearn_C-Systems_Resume.pdf` — **Set**
    

If you want, paste a JD you plan to apply to and I’ll tailor the Skills lines and 3–5 bullets to match it tightly (that’s the single biggest ATS boost).