Related to [[Embedded]], [[Hollix]] 
#embedded #embedded_project_phases #hollix 

| Phases                         | Modules                                                                            | Approx LOC   | %Of the Total Work |     |     |
| ------------------------------ | ---------------------------------------------------------------------------------- | ------------ | ------------------ | --- | --- |
| **0 — Platform** (you're here) | systick, timers, watchdog, logging, ringbuf                                        | ~400         | ~5%                |     |     |
| **1 — HAL Abstraction**        | GPIO, UART driver, SPI/I2C driver, button, LED patterns, ADC                       | ~1,500–2,500 | ~25%               |     |     |
| **2 — Application Logic**      | State machines, sensor processing, command parser, data formatting, business rules | ~3,000–5,000 | ~50%               |     |     |
| **3 — Robustness**             | Fault handlers, error recovery, assert framework, safe defaults                    | ~500–1,000   | ~10%               |     |     |
| **4 — Polish**                 | Power management, config in flash, production log tuning                           | ~500–800     | ~10%               |     |     |
|                                |                                                                                    |              |                    |     |     |

At that point you're past firmware development and into **product engineering**:

### Phase 5: Testing & Verification

- **Unit tests** on host (compile your modules with GCC on Linux, mock the HAL) `App/`
- **Integration tests** on target (automated via UART commands + Python scripts)
- **Static analysis** (MISRA-C compliance, `cppcheck`, Clang's `scan-build`)
- **Code coverage** — prove which paths are actually exercised
- **Regression suite** — CI runs tests on every commit

### Phase 6: Build & Release Infrastructure

- **CI/CD pipeline** (GitHub Actions / GitLab CI building both Debug and Release)
- **Versioning** — semantic version baked into firmware, queryable over UART
- **Binary artifacts** — `.bin`, `.hex`, published per release `.elf`
- **OTA/bootloader** — if the product needs field updates, you write a bootloader that lives in the first flash sector and jumps to your app
- **Release notes** — changelog tied to git tags

### Phase 7: Manufacturing & Provisioning

- **Factory flash scripts** — program + verify in one step (OpenOCD / STM32CubeProgrammer CLI)
- **Per-device calibration** — store calibration data in a reserved flash page
- **Unique device ID / serial numbers** — read from STM32's UID registers, stored or printed
- **End-of-line test jig** — automated hardware test that runs on every board off the line

### Phase 8: Field Support & Maintenance

- **Crash/fault logging** — `HardFault_Handler` captures stack frame to flash, readable on next boot
- **Telemetry** — runtime health metrics (uptime, error counts, min/max values)
- **Field firmware updates** — whatever mechanism you built in Phase 6, now customers are using it
- **Bug triage** — reproduce from field logs, fix, regression test, release patch