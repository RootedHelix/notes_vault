Related to [[Hollix]], [[Embedded]], [[Hardware compatibility-matrix]]

# Hollix current steps — 2026-11-19

The NUCLEO-H753ZI has been ordered. Hardware compatibility has been verified in [[Hardware compatibility-matrix]], including RGB565/LTDC pin mapping, ST7701S initialization, display timing, backlight requirements, and TPS61176EVM capability.

The next phase has two parallel workstreams: the KiCad interposer and STM32 firmware.

## Track 1 — KiCad LCD interposer

1. Inspect the physical display:
   - Confirm the panel revision.
   - Confirm FPC pin 1.
   - Confirm which side exposes the FPC contacts.

2. Select the 45-pin, 0.5 mm-pitch connector:
   - `DM-ACC05045-908` for top contact, or
   - `DM-ACC05045-909` for bottom contact.
   - Match the selected connector to the ribbon entry direction and PCB placement.
   - Verify support for the 0.30 ± 0.03 mm stiffened FPC tail.

3. Create the interposer schematic using [[Hardware compatibility-matrix]] as the source of truth:
   - 16 RGB565 data lines.
   - `PCLK`, `HSYNC`, `VSYNC`, and `DE`.
   - 3-wire SPI and reset.
   - `VCI`, `IOVCC`, ground, and local decoupling.
   - Separate `LEDA` and `LEDK` backlight connections.
   - CN11/CN12 connections to the Nucleo.
   - Configuration for unused RGB data pins.

4. Assign exact footprints and verify:
   - Connector pin numbering and contact side.
   - Nucleo header orientation.
   - Decoupling placement.
   - Backlight connector/wiring.

5. Lay out the PCB:
   - Keep RGB and clock traces short.
   - Use a continuous ground-return plane.
   - Keep the PCLK path clean.
   - Avoid long jumper-wire connections.

6. Run KiCad ERC and DRC.

7. Do not release the PCB for fabrication until the physical panel revision, connector orientation, connector footprint, and pin-1 mapping have been checked.

## Track 2 — STM32H753 display firmware

1. Prepare the ARM toolchain, CMake project, linker script, and ST-LINK/OpenOCD debugging.

2. When the Nucleo arrives:
   - Flash a basic GPIO program.
   - Verify breakpoints and debugging.
   - Confirm the board revision and required solder-bridge changes.
   - Disconnect the PB0 user LED before using PB0 as `LCD_R3`.

3. Implement the display control interface:
   - Active-low hardware reset and required delays.
   - 3-wire, 9-bit SPI.
   - Port the supplied vendor initialization sequence exactly.

4. Configure LTDC:
   - 480 × 480 active area.
   - RGB565.
   - Approximately 17 MHz pixel clock.
   - Active-high `DE`.
   - Active-low `HSYNC` and `VSYNC`.
   - Pixel data sampled by the display on PCLK's falling edge.

5. Implement the framebuffer:
   - Place one 460,800-byte RGB565 framebuffer in AXI SRAM.
   - Configure MPU/cache behavior correctly.
   - Add pixel, fill, and rectangle operations.
   - Add red, green, blue, white, black, and checkerboard test patterns.

6. Implement backlight control:
   - Generate PWM for the TPS61176EVM.
   - Use two combined 20 mA sink channels for 40 mA total.
   - Default the backlight to off during reset and display initialization.

## Hardware bring-up after the interposer arrives

1. Assemble without enabling the backlight.
2. Verify continuity, ground, `VCI`, and `IOVCC`.
3. Confirm reset and 3-wire SPI waveforms.
4. Run the ST7701S initialization sequence.
5. Verify LTDC timing and test patterns.
6. Connect `LEDA` to TPS61176EVM `VOUT`.
7. Connect `LEDK` to the two combined enabled IFB channels.
8. Begin at low PWM duty and measure voltage/current.
9. Increase toward 40 mA total only after stable image and power operation are confirmed.

## Immediate actions

- Begin the KiCad interposer schematic.
- Begin the STM32 firmware project in parallel.
- Inspect the physical display and choose the connector orientation before PCB layout.
