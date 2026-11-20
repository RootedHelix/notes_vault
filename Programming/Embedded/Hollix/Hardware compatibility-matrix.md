Related to [[Hollix]], [[Embedded]]

# Hollix display / NUCLEO-H753ZI compatibility matrix

## Documents reviewed

- `display_datasheet.pdf` — DisplayModule DM-TFT40-388, 4.0-inch 480 × 480 IPS display, 17 pages, 2019 second release.
- `DS_stm32h753zi.pdf` — STM32H753xI datasheet, DS12117 Rev 10, 357 pages.
- `um2407-stm32h7-nucleo144-board-stmicroelectronics.pdf` — STM32H7 Nucleo-144 board user manual, UM2407 Rev 6, 54 pages.
- `DM-TFT40-388_20Controller_20Datasheet.pdf` — Sitronix ST7701S controller specification, Version 1.2, 308 pages.
- `DM-TFT40-388_20initialization_20code.rar` — DisplayModule package containing interface notes and initialization code for 16-, 18-, and 24-bit RGB modes. The RGB565 findings below use `4inch 480480 Initialized code-3SPI-16BIT RGB.txt` and `4inch 480480-interface-3SPI-16BIT RGB.pdf` from this archive.
- `dataSheet_tps61176.pdf` — Texas Instruments TPS61176 six-channel WLED driver datasheet, SLVSBM5B.
- `User_guide_slvu998.pdf` — Texas Instruments TPS61176EVM-566 user's guide, SLVU998.

Page references below are PDF page numbers. These PDFs also print the same page number in their footers. The initialization source is a text file and therefore has no page numbers; function and command names are cited instead.

## Conclusion

**Prototype architecture pass:** the STM32H753ZIT6 on the NUCLEO-H753ZI can generate the display's required 16-bit RGB interface, a complete RGB565 pin set is exposed on CN11/CN12, and the TPS61176EVM can supply the required 18.6 V / 40 mA backlight load by combining two 20 mA sink channels.

The proposed connection uses:

- 16 RGB data lines
- `PCLK`, `HSYNC`, `VSYNC`, and `DE`
- A separate serial control interface
- A reset GPIO
- 3.3 V panel and logic supplies
- An external constant-current boost driver for the backlight

The available documentation now resolves the serial framing, RGB mode, signal polarities, pixel format, initialization sequence, backlight operating point, and backlight-driver capability. Before releasing the KiCad interposer for fabrication:

1. Inspect the physical panel and confirm its revision marking, FPC pinout, pin-1 location, and contact side match display datasheet p. 5.
2. Select a top- or bottom-contact connector based on its placement in KiCad and the actual ribbon entry direction.
3. Verify the selected connector's manufacturer footprint, pin-1 orientation, actuator clearance, 45-position/0.5 mm pitch, and support for the 0.30 ± 0.03 mm stiffened FPC tail.
4. Archive the supplied initialization package with the firmware documentation. The initialization code is not itself a PCB-fabrication blocker once the physical panel revision and pinout are confirmed.

## Display requirements and source locations

| Requirement | Finding | Where to find it |
|---|---|---|
| Module | DM-TFT40-388, ST7701S controller | Display datasheet p. 3, **2 Main Features** |
| Resolution | 480(RGB) × 480 | Display datasheet p. 3, **2 Main Features** |
| Host interface | 16-, 18-, or 24-bit parallel RGB | Display datasheet p. 3, **2 Main Features** |
| Connector | 45 contacts, 0.50 mm pitch | Display datasheet p. 5, **4 Mechanical Drawing** |
| FPC orientation | Drawing identifies the exposed **CONTACT SIDE** and shows pins 45 and 1 | Display datasheet p. 5, **4 Mechanical Drawing** |
| Panel supply | `VCI`: 2.5–3.6 V, 3.3 V typical | Display datasheet p. 6, **5.3 DC Characteristics** |
| Logic supply | `IOVCC`: 1.65–3.3 V, 1.8 V typical | Display datasheet p. 6, **5.3 DC Characteristics** |
| Panel current | `ICC`: 30 mA typical, excluding the backlight | Display datasheet p. 6, **5.3 DC Characteristics** |
| Logic levels | Low ≤ 0.3 × IOVCC; high ≥ 0.7 × IOVCC | Display datasheet p. 6, **5.3 DC Characteristics** |
| Backlight terminals | Pin 43 `LEDK`, pin 45 `LEDA` | Display datasheet p. 4, **3.1 TFT** |
| Backlight rating | 18.6 V typical, 40 mA typical; 12 white LED chips | Display datasheet pp. 5 and 7, mechanical notes and **5.4 LED Backlight Characteristics** |
| Rated luminance | 950 cd/m² minimum, 1000 cd/m² typical | Display datasheet p. 7, **5.4 LED Backlight Characteristics** |
| Pixel clock | 17 MHz typical for approximately 60 Hz | Display datasheet p. 14, **6.2 RGB Interface Definition** |
| Horizontal timing | Sync 8, back porch 50, active 480, front porch 10 clocks | Display datasheet p. 14, **6.2 RGB Interface Definition** |
| Vertical timing | Sync 8, back porch 20, active 480, front porch 10 lines | Display datasheet p. 14, **6.2 RGB Interface Definition** |
| RGB electrical timing | PCLK period ≥ 33 ns; high/low pulse ≥ 15 ns; data setup/hold ≥ 5 ns | Display datasheet p. 10, **5.5.3 RGB Interface Characteristics** |
| Reset | Active low; pulse ≥ 10 μs; wait at least 5 ms before commands; Sleep Out cannot be sent for 120 ms | Display datasheet p. 11, **5.5.4 Reset input timing** |
| Serial timing | Write clock period ≥ 66 ns, equivalent to ≤ 15.15 MHz | Display datasheet pp. 8–9, **5.5.1** and **5.5.2** |
| Interface mode | RGB operation must be configured through the serial interface | Display datasheet pp. 5, 12, and 13 |

The typical totals are:

- Horizontal total: 8 + 50 + 480 + 10 = **548 clocks**
- Vertical total: 8 + 20 + 480 + 10 = **518 lines**
- At 60 Hz: 548 × 518 × 60 = **17.032 MHz**, consistent with the stated 17 MHz typical clock.

## Display FPC pin groups

The authoritative pin table is on display datasheet p. 4, **3.1 TFT**. The mechanical drawing and contact-side orientation are on p. 5.

| FPC pins | Function |
|---|---|
| 1–4 | Resistive-touch terminals; the feature table says no touch panel is fitted, so confirm whether these are physically connected |
| 5–6 | Ground |
| 7 | `VCI`, panel supply |
| 8 | `IOVCC`, logic supply |
| 9 | `SDO`; datasheet says it may be left open when unused |
| 10 | `SDI`, serial data |
| 11 | `SCL`, serial clock |
| 12 | `CS`, serial chip select |
| 13 | Active-low `RESET` |
| 14–37 | `DB23` through `DB0` |
| 38 | `DE` |
| 39 | `PCLK` |
| 40 | `HSYNC` |
| 41 | `VSYNC` |
| 42 | Not connected |
| 43 | `LEDK` |
| 44 | Not connected |
| 45 | `LEDA` |

### FPC verification before choosing the connector

On display datasheet p. 5, inspect the enlarged FPC tail:

- It labels the **CONTACT SIDE**.
- It shows pin 45 at one end and pin 1 at the other when viewed as drawn.
- It specifies 45 contacts at 0.50 mm pitch and a 23.00 mm tail width.
- It identifies the stiffened insertion tail as **0.30 ± 0.03 mm** thick; the unstiffened FPC is shown separately as 0.10 ± 0.03 mm.

DisplayModule's online connector table lists two matching catalog options:

- [`DM-ACC05045-908`](https://www.displaymodule.com/pages/product-table-list): 45 positions, 0.5 mm pitch, top-contact, drawer lock.
- [`DM-ACC05045-909`](https://www.displaymodule.com/pages/product-table-list): 45 positions, 0.5 mm pitch, bottom-contact, flip lock.

The panel does not electrically require top versus bottom contact. Select the version that places the connector contacts against the FPC's labeled **CONTACT SIDE** in the intended PCB/ribbon geometry. Obtain the chosen connector's manufacturer drawing and verify its footprint, pin-1 direction, actuator clearance, and acceptance of a 0.30 mm FPC before laying out the board.

## Proposed RGB565 mapping

The display's required 16-bit bus selection is on display datasheet p. 5, **4 Mechanical Drawing**, and p. 13, **6.1 RGB Color Format**:

- Red uses display `DB20`–`DB16`.
- Green uses display `DB13`–`DB8`.
- Blue uses display `DB4`–`DB0`.

The unused display data pins are `DB23`–`DB21`, `DB15`–`DB14`, and `DB7`–`DB5`. The pin description on display datasheet p. 4 says unused data pins must be fixed to `IOVCC` or ground. Grounding them through individual 0 Ω configuration resistors is a practical option.

STM32 LTDC alternate functions are listed in MCU datasheet pp. 89–100, **Table 9: Alternate functions**. Nucleo header positions are listed in Nucleo manual pp. 43–44, **Table 22: Pin assignment of the ST morpho connector**.

| Display FPC | Display signal | STM32 LTDC output | STM32 pin | AF | Nucleo header |
|---:|---|---|---|---:|---|
| 17 | `DB20` / R4 | `LCD_R7` | PE15 | AF14 | CN12-53 |
| 18 | `DB19` / R3 | `LCD_R6` | PB1 | AF14 | CN12-24 |
| 19 | `DB18` / R2 | `LCD_R5` | PC0 | AF14 | CN11-38 |
| 20 | `DB17` / R1 | `LCD_R4` | PA5 | AF14 | CN12-11 |
| 21 | `DB16` / R0 | `LCD_R3` | PB0 | AF14 | CN11-34 |
| 24 | `DB13` / G5 | `LCD_G7` | PD3 | AF14 | CN11-40 |
| 25 | `DB12` / G4 | `LCD_G6` | PC7 | AF14 | CN12-19 |
| 26 | `DB11` / G3 | `LCD_G5` | PB11 | AF14 | CN12-18 |
| 27 | `DB10` / G2 | `LCD_G4` | PB10 | AF14 | CN12-25 |
| 28 | `DB9` / G1 | `LCD_G3` | PE11 | AF14 | CN12-56 |
| 29 | `DB8` / G0 | `LCD_G2` | PA6 | AF14 | CN12-13 |
| 33 | `DB4` / B4 | `LCD_B7` | PB9 | AF14 | CN12-5 |
| 34 | `DB3` / B3 | `LCD_B6` | PB8 | AF14 | CN12-3 |
| 35 | `DB2` / B2 | `LCD_B5` | PA3 | AF14 | CN12-37 |
| 36 | `DB1` / B1 | `LCD_B4` | PE12 | AF14 | CN12-49 |
| 37 | `DB0` / B0 | `LCD_B3` | PA8 | AF13 | CN12-23 |
| 38 | `DE` | `LCD_DE` | PE13 | AF14 | CN12-55 |
| 39 | `PCLK` | `LCD_CLK` | PE14 | AF14 | CN12-51 |
| 40 | `HSYNC` | `LCD_HSYNC` | PC6 | AF14 | CN12-4 |
| 41 | `VSYNC` | `LCD_VSYNC` | PA4 | AF14 | CN11-32 |

The reason the STM32's upper color bits connect to differently numbered display bus bits is that the STM32 expands RGB565 framebuffer values onto the upper bits of its 24-bit LTDC output, while the display's 16-bit mode receives compact five/six/five-bit values on the particular `DB` pins listed above.

### Board conflict requiring action

PB0 is connected to the Nucleo green user LED in the default board configuration. PB0 is also the only practical exposed `LCD_R3` choice in this mapping.

- See Nucleo manual pp. 31–32, **Table 14**, entries `SB39, SB47 (LD1-LED)`.
- Disconnect LD1 from PB0 before relying on PB0 as an LTDC data signal. Follow Table 14 for the actual solder-bridge state on the purchased board revision.
- Do not reroute LD1 to PA5, because PA5 is used here as `LCD_R4`; disconnecting LD1 entirely is the clean choice.

The selected mapping avoids the pins used by the onboard Ethernet RMII interface and avoids using PA11/PA12 for LTDC, so USB can remain available. Nevertheless, verify the actual solder-bridge state against Nucleo manual pp. 31–35, **Table 14**, before assembly.

## Serial-control and reset mapping

The module-specific source confirms that this panel uses the ST7701S **3-wire, 9-bit serial protocol**. Each transfer consists of:

1. One D/C control bit, transmitted first (`0` for command, `1` for parameter data).
2. Eight payload bits, MSB first.
3. The controller samples SDA on SCL's rising edge.
4. `CS` is active low and may return high between command and parameter words.

References:

- ST7701S controller datasheet p. 58, **8.2.1.1 Command write mode** and **Figure 10**.
- ST7701S controller datasheet p. 59, **Figure 11: 3-line serial interface write protocol**.
- Initialization source functions `write_command()` and `write_data()`, which implement the leading D/C bit followed by eight data bits.

A non-conflicting SPI4 allocation is:

| Display FPC | Signal | STM32 pin | Nucleo header | Notes |
|---:|---|---|---|---|
| 9 | `SDO` | PE5 / SPI4_MISO | CN11-50 | Optional; leave open for write-only bring-up |
| 10 | `SDI` | PE6 / SPI4_MOSI | CN11-62 | Must transmit 9-bit words, D/C bit first |
| 11 | `SCL` | PE2 / SPI4_SCK | CN11-46 | Use SPI mode with sampling on the rising edge; begin at or below 10 MHz |
| 12 | `CS` | PE4 / GPIO or SPI4_NSS | CN11-48 | Active low; software-controlled CS is simplest |
| 13 | `RESET` | PE3 / GPIO | CN11-47 | Active low |

Confirm the SPI4 alternate functions in MCU datasheet pp. 89–100, **Table 9**, and the physical header pins in Nucleo manual pp. 43–44, **Table 22**. If the STM32 SPI configuration cannot produce the required 9-bit framing cleanly, transmit it with GPIO bit-banging as the vendor source does.

### Confirmed initialization sequence

The authoritative module-specific sequence is in `4inch 480480 Initialized code-3SPI-16BIT RGB.txt`, function `initi()`.

Key operations are:

- Hardware reset: RESET high for 1 ms, low for 10 ms, then high for 200 ms.
- Send Sleep Out (`11h`) and wait 120 ms.
- Select ST7701S Command 2 BK0 with `FFh: 77 01 00 00 10`.
- Program display-line, porch, inversion, RGB control, and positive/negative gamma registers.
- Select Command 2 BK1 with `FFh: 77 01 00 00 11` and program the panel's analog power, source timing, and GIP registers.
- Select Command 2 BK3, write the supplied VAP/VAN setting, then return to BK0.
- Set MADCTL (`36h`) to `00h`.
- Set COLMOD (`3Ah`) to `50h`, selecting RGB565. ST7701S controller datasheet p. 218, **12.2.30 COLMOD**, defines `VIPF=101` as 16-bit/pixel.
- Send Display On (`29h`) and wait 25 ms.

Preserve the vendor's complete register byte sequence exactly in firmware; the gamma, GIP, VCOM, VGH/VGL, AVDD, and AVCL values are panel-specific and should not be reconstructed from this summary.

## Timing compatibility

### Display

Display datasheet p. 14 gives the typical 17 MHz pixel clock and porch/sync values. Display datasheet p. 10 allows a minimum PCLK period of 33 ns, corresponding to a maximum of approximately 30.3 MHz.

### STM32H753

- The MCU includes a 24-bit parallel RGB LTDC supporting panels up to 1024 × 768: MCU datasheet p. 40, **3.25 LCD-TFT controller**.
- LTDC electrical timing is specified on MCU datasheet pp. 185–186 for revision Y and pp. 295–296 for revision V.
- MCU datasheet p. 185, **Table 91**, specifies LTDC clock capability far above the display's 17 MHz requirement; even the lowest listed condition permits 90 MHz.

Therefore, pixel-clock frequency and resolution are comfortably compatible. Configure the initial timing as:

- Active width: 480
- Active height: 480
- HSYNC width: 8
- Horizontal back porch: 50
- Horizontal front porch: 10
- VSYNC width: 8
- Vertical back porch: 20
- Vertical front porch: 10
- Pixel clock: approximately 17.0 MHz

The module-specific initialization code resolves the signal configuration. It writes `C3h` with first parameter `02h`. According to ST7701S controller datasheet p. 272, **12.3.2.10 RGBCTRL**:

- `DE/HV=0`: DE mode.
- `VSP=0`: VSYNC active low.
- `HSP=0`: HSYNC active low.
- `DP=1`: pixel data is sampled on DOTCLK's negative/falling edge.
- `EP=0`: RGB data is valid while DE/ENABLE is high.

Configure the STM32 LTDC output to produce those polarities. In DE mode, HSYNC, VSYNC, PCLK, and DE are all still required; see ST7701S controller datasheet p. 72, **8.6 RGB Interface**.

## Framebuffer memory

An RGB565 framebuffer requires:

`480 × 480 × 2 = 460,800 bytes`

Two complete buffers require:

`921,600 bytes`

Relevant MCU references:

- MCU datasheet p. 1: 1 MB total RAM, including TCM and user SRAM.
- MCU datasheet p. 26, **3.3.3 Embedded SRAM**: 512 KB AXI SRAM, 128 KB SRAM1, 128 KB SRAM2, 32 KB SRAM3, and 64 KB SRAM4.
- MCU datasheet p. 40, **3.25 LCD-TFT controller**: the LTDC uses an AXI master interface.

One 460,800-byte framebuffer fits in the 512 KB AXI SRAM, leaving only 63,488 bytes there. Two buffers do not fit together in AXI SRAM. Double buffering would require a carefully verified multi-bank arrangement or external RAM. The LTDC framebuffer must not be placed blindly in TCM; use the reference manual and bus-access rules when preparing the linker script.

## Power compatibility

### Panel logic

Operating the Nucleo and display at 3.3 V is electrically compatible:

- Display `VCI`: 2.5–3.6 V.
- Display `IOVCC`: 1.65–3.3 V.
- Nucleo I/O is 3.3 V compatible, not 5 V compatible: Nucleo manual p. 39, cautions below **Figure 16**.
- Nucleo 3.3 V and ground are available on the Morpho headers: Nucleo manual p. 43, **Table 22**.

The display logic consumes approximately 30 mA typical, but local decoupling is still required at FPC pins 7 and 8. Use separate decoupling networks for `VCI` and `IOVCC`, even if both are fed from 3.3 V.

### Backlight operating point — resolved

The backlight must not be powered from a Nucleo GPIO or directly from its 3.3 V rail. Its rated terminal operating point is:

- `VLED = 18.6 V` typical.
- `ILED = 40 mA` total at FPC pin 45 (`LEDA`) and pin 43 (`LEDK`).
- Electrical output power: `18.6 V × 0.040 A = 0.744 W` before driver losses.

Evidence:

1. Display datasheet p. 5 shows 12 white LEDs arranged as **two parallel branches of six series LEDs**, and labels the backlight 18.6 ± 0.3 V, 40 mA.
2. Display datasheet p. 7 gives 18.6 V and 40 mA typical, with lifetime characterized at 40 mA.
3. The vendor's `4inch 480480-interface-3SPI-16BIT RGB.pdf` explicitly states `VLED=18.6V, ILED=40mA`.
4. The [DM-TFT40-388 product page](https://www.displaymodule.com/products/4-inch-ips-high-brightness-display-480x480-1000nits-with-rgb) specifies 1000 cd/m² and describes the backlight as 18.6 V typical with 50,000-hour life at 40 mA.

The `(IF = 20mA)` text beside luminance on display datasheet p. 7 refers to the current in each internal six-LED branch, or is abbreviated wording for that per-branch condition. Two approximately 20 mA branches produce the specified 40 mA total at the common `LEDA/LEDK` terminals.

### TPS61176EVM-566 compatibility — pass with combined channels

The TPS61176 is suitable for this load:

- TPS61176 datasheet p. 1: six current sinks, each capable of up to 35 mA, with a boost output suitable for the 18.6 V string voltage.
- TPS61176 datasheet pp. 22–23, **8.2.3 Combined String Application**: two or three sink channels may be combined for higher-current loads; Figure 19 shows up to 60 mA per combined string.
- TPS61176EVM guide p. 5: JP1–JP6 individually enable or disable IFB channels.
- TPS61176EVM guide p. 7: the EVM is characterized at 20 mA per channel.
- TPS61176EVM guide p. 10, **Figure 9**: identifies `VOUT`, `IFB1`–`IFB6`, channel jumpers, and the default current-setting circuit.

Recommended prototype connection:

1. Connect display `LEDA` to EVM `VOUT`.
2. Combine two IFB sink channels, such as `IFB1` and `IFB2`, and connect the combined node to display `LEDK`.
3. Set each enabled channel to 20 mA, producing 40 mA aggregate current.
4. Enable the two used channels with their jumpers and disable the other four channels according to EVM guide p. 5.
5. Begin with a reduced PWM duty cycle, verify VOUT and total LED current with instruments, and only then test 100% duty.

This connection uses the driver's documented combined-channel mode; one 40 mA sink channel must not be used because the per-channel limit is 35 mA.

## Resolved documentation ambiguities

### DE polarity

The module datasheet p. 4 says DE low is enabled, contradicting its own RGB timing sections. The controller datasheet and module-specific initialization resolve this in favor of **active-high DE**:

- Controller datasheet p. 72 says RGB Mode 1 writes data while DE is high.
- Controller datasheet p. 272 defines `EP=0` as data valid while ENABLE is high.
- The initialization source writes `C3h = 02h`, which sets `EP=0`.

Treat the p. 4 module pin-description wording as an error.

### Initialization and clock polarity

The initialization package supplies the previously missing panel-specific register sequence. Its `C3h = 02h` setting also confirms DE mode, active-low sync signals, and falling-edge PCLK sampling. These are no longer open compatibility questions, but they must be reproduced exactly in firmware.

### Revision confirmation

The display PDF is from 2019. Compare the part/revision markings printed on the purchased panel's flex cable with `DM-TFT40-388` and obtain written vendor confirmation that the current hardware still uses the p. 4/p. 5 pinout.

## Purchase/design decision

The NUCLEO-H753ZI is a viable development platform for this display, subject to:

- Disconnecting the PB0 user LED load;
- Implementing the supplied 3-wire/9-bit initialization sequence exactly;
- Configuring active-high DE, active-low HSYNC/VSYNC, and falling-edge pixel sampling;
- Configuring two TPS61176 sink channels for 20 mA each and verifying 40 mA total before full-brightness operation;
- Selecting either the top-contact `DM-ACC05045-908` or bottom-contact `DM-ACC05045-909` to match the PCB/ribbon geometry, then verifying its manufacturer footprint; and
- Keeping the RGB connection short and providing a continuous ground-return path.

The interposer schematic should not be released for fabrication until the connector orientation/footprint and physical panel revision are checked.

## External source links

- [DisplayModule DM-TFT40-388 product page](https://www.displaymodule.com/products/4-inch-ips-high-brightness-display-480x480-1000nits-with-rgb) — resolution, luminance, controller, interface, backlight voltage/current context, and downloadable resources.
- [DisplayModule product and connector table](https://www.displaymodule.com/pages/product-table-list) — DM-TFT40-388 listing and `DM-ACC05045-908` / `DM-ACC05045-909` connector options.
- [Texas Instruments TPS61176 datasheet](https://www.ti.com/lit/ds/symlink/tps61176.pdf) — current-sink limits and combined-channel application.
- [Texas Instruments TPS61176EVM-566 user's guide](https://www.ti.com/lit/ug/slvu998/slvu998.pdf) — EVM connections, jumpers, current setting, and schematic.
