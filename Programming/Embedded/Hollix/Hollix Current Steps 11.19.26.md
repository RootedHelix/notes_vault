Related to [[Hollix]], [[Embedded]], [[Hardware compatibility-matrix]]

# Hollix current steps — 2026-11-19

The NUCLEO-H753ZI is on hand and is the MCU development board being transitioned to. Hardware compatibility has been verified in [[Hardware compatibility-matrix]], including RGB565/LTDC pin mapping, ST7701S initialization, display timing, backlight requirements, and TPS61176EVM capability.

The next phase has two parallel workstreams: the KiCad interposer and STM32 firmware.

## Track 1 — KiCad LCD interposer

1. Inspect the physical display — **complete for PCB development**:
   - The order record identifies the purchased supplier product as `DM-TFT40-388`; it does not list a separate hardware revision.
   - Manufacturer panel model printed on the physical display: `KD040WXFPD014`.
   - Additional rear traceability codes: `930460330A` and `07E5G10A`.
   - Physical white FPC marking: `40014 V0 FPC40B015 HXD 1910`.
   - `V0` appears to be the FPC revision; it is not the assembled-module or datasheet revision.
   - Public Startek listings identify `KD040WXFPD014` as the non-touch 4-inch, 480 × 480, ST7701S, 1000-nit RGB module matching `DM-TFT40-388`. No conflicting touch-panel suffix is printed on this unit.
   - The available online V1.2 specification has document revisions V1.0 (2019-03-14, first issue), V1.1 (2019-04-30, drawing update), and V1.2 (2019-07-10, corrections). These are specification-document revisions, not panel hardware revisions.
   - The local DisplayModule datasheet records a first release on 2019-05-17 and a second release on 2019-07-16. These are also document releases. [Source: `hollix_data_sheets/display_datasheet.pdf`, PDF p. 3.]
   - No public source maps `V0`, `930460330A`, or `07E5G10A` to a separate assembled-module revision, and no additional customer-visible hardware revision was found.
   - FPC pin 45 is marked beside one edge of the contact array; pin 1 is at the opposite edge and is confirmed by the mechanical drawing.
   - The exposed gold contacts are on the rear-facing side shown in the physical-display photograph.
   - Intended installation: active display face downward, with the metal rear and exposed FPC contacts upward.
   - Intended interposer placement: above/behind the LCD near the FPC exit, with the ribbon flat and untwisted.
   - Supplier confirmation is optional for procurement traceability and remains a pre-fabrication safeguard, not a blocker for PCB development.
   - Online references:
     - Startek module listing: https://www.touch-tft.com/4-inch-480x480-st7701s-rgb-interface-square-tft-high-brightness-wide-temperature-ips-lcd.html
     - Product/specification listing: https://www.top-electronics.com/en/4-0-ips-480-480-1000nits-rgb
     - V1.2 specification and revision history: https://www.panoxdisplay.com/uploadfile/datasheet/(4.0LCD)PL040F480N01.pdf

2. Select the 45-pin, 0.5 mm-pitch connector — **mechanical drawing obtained; final checks pending**:
   - Selected supplier SKU: top-contact `DM-ACC05045-908` for the intended face-down display and rear-mounted interposer arrangement.
   - The provided connector drawing identifies the manufacturer as Shenzhen Holy Electronic Co., Ltd. and the drawing/series as `FPC0520SQ-nP`. [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - Connector type is a 0.5 mm-pitch, 2.0 mm-high SMT ZIF described as `UPPER CONN`, confirming top-contact orientation. [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - The drawing includes both the recommended FPC/FFC dimensions and the recommended PCB land pattern. [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - For `n = 45`, the contact span is `0.5 × (45 − 1) = 22.00 mm`; the drawing's related layout span is `1.0 + 0.5 × (45 − 1) = 23.00 mm`. [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - Connector recommended FPC thickness is `0.30 +0.03/−0.02 mm` (0.28–0.33 mm). [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - Display stiffener thickness is `0.30 ± 0.03 mm` (0.27–0.33 mm). [Source: `hollix_data_sheets/display_datasheet.pdf`, PDF p. 5.]
   - The nominal thicknesses agree, but their minimum tolerances differ by 0.01 mm. Retain DisplayModule's connector recommendation as compatibility evidence or request explicit confirmation that `DM-ACC05045-908` is qualified for the display's full stiffener tolerance.
   - Connector ratings include 50 V AC/DC, 0.5 A AC/DC, contact resistance ≤20 mΩ, and operating temperature −25 °C to +85 °C. [Source: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.]
   - The display has 45 FPC positions. Pin 1 is `XR`, pin 45 is `LEDA`, and its drawing marks both ends. [Source: `hollix_data_sheets/display_datasheet.pdf`, PDF pp. 4–5.]
   - **Still required before final footprint approval:** visually identify terminal 1 on the connector's recommended PCB layout, map it to KiCad pad 1, and compare a 1:1 footprint printout with the connector/FPC.

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
   - Place the interposer near the FPC exit so the ribbon remains flat and untwisted.
   - Use insulated spacers between the interposer and the LCD metal backing, and provide FPC strain relief.
   - Keep the interposer and wiring outside the optical area and within the top enclosure.
   - Keep RGB and clock traces short.
   - Use a continuous ground-return plane.
   - Keep the PCLK path clean.
   - Avoid long jumper-wire connections.

6. Run KiCad ERC and DRC.

7. Do not release the PCB for fabrication until the physical panel revision, connector orientation, connector footprint, and pin-1 mapping have been checked.

## Track 2 — STM32H753 display firmware

1. Prepare the ARM toolchain, CMake project, linker script, and ST-LINK/OpenOCD debugging.

2. Bring up the Nucleo:
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
   - Add a mirrored-image test because the angled reflective display arrangement may horizontally reverse the viewed image; apply framebuffer or artwork mirroring if testing confirms it.

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
- Create the `DM-ACC05045-908` KiCad footprint from the provided mechanical drawing; verify terminal 1 and check a 1:1 printout against the connector/FPC before PCB layout.
- Prototype the reflection angle, image mirroring, brightness, and optical alignment before finalizing the enclosure.
