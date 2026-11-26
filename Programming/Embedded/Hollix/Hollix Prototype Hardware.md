Related to [[Hollix]], [[Hollix_Design]]
#hollix #embedded #hollix_prototype_hardware

## Current Hardware


### DISPLAY - 4.0" IPS High Brightness Display 480x480 1000nits - RGB

- Supplier product model: `DM-TFT40-388` (non-touch)
- Manufacturer panel model: `KD040WXFPD014`
- Controller: `ST7701S`
- Interface: parallel RGB pixel data with serial controller configuration
- Module dimensions: 77.0 × 80.0 × 2.3 mm
- Active area: 71.86 × 70.18 mm
- Typical supplies: `VCI` 3.3 V and `IOVCC` 1.8 V
- Backlight reference: 18.6 V at 40 mA typical
- Product link: https://www.displaymodule.com/products/4-inch-ips-high-brightness-display-480x480-1000nits-with-rgb?variant=20413601579106&country=US&currency=USD&utm_source=chatgpt.com
- Data sheets and technical information are included on the product webpage.

#### Physical display identification

- Manufacturer panel model printed on the rear: `KD040WXFPD014`
- Additional rear traceability codes visible: `930460330A` and `07E5G10A`
- White marking printed on the FPC: `40014 V0 FPC40B015 HXD 1910`
- `V0` appears to identify the FPC revision; it is not necessarily the complete module or datasheet revision.
- Published information for `KD040WXFPD014` confirms a 45-pin, 0.5 mm-pitch FPC and matches the purchased display specifications.
- The FPC has `45` printed beside one edge of the contact array; that edge is pin 45 and pin 1 is at the opposite edge. Verify the orientation against the module drawing before assigning footprint pins.
- The exposed gold FPC contacts are on the rear-facing side shown in the physical-display photograph.
- The other rear codes appear to be manufacturing or lot traceability identifiers rather than PCB design parameters.

#### Purchase and revision verification

- The order record lists only `DM-TFT40-388`, which is the DisplayModule supplier part number.
- The physical panel is marked `KD040WXFPD014`, the Startek manufacturer part number for the matching non-touch 4-inch, 480 × 480, ST7701S, 1000-nit RGB module.
- No touch-panel suffix is printed on this unit, which agrees with the ordered non-touch `DM-TFT40-388` variant.
- The local DisplayModule datasheet records a first release on 2019-05-17 and a second release on 2019-07-16. [Source: `hollix_data_sheets/display_datasheet.pdf`, PDF p. 3.]
- The available online V1.2 specification lists V1.0 (2019-03-14, first issue), V1.1 (2019-04-30, drawing update), and V1.2 (2019-07-10, corrections). These are specification-document revisions, not assembled-panel hardware revisions.
- Searches for the exact FPC and rear traceability codes did not identify another public hardware revision. `V0` remains recorded as the apparent FPC revision.
- The order record, manufacturer marking, physical dimensions, interface, and electrical specifications agree. Physical identification is complete for PCB development.
- Supplier confirmation is optional for procurement traceability and may remain a final pre-fabrication safeguard.
- Online references:
  - Startek module listing: https://www.touch-tft.com/4-inch-480x480-st7701s-rgb-interface-square-tft-high-brightness-wide-temperature-ips-lcd.html
  - Product/specification listing: https://www.top-electronics.com/en/4-0-ips-480-480-1000nits-rgb
  - V1.2 specification and revision history: https://www.panoxdisplay.com/uploadfile/datasheet/(4.0LCD)PL040F480N01.pdf

- Intended installation: active display face downward, with the metal rear and exposed FPC contacts facing upward.
- Intended interposer placement: above/behind the LCD near the FPC exit, keeping the ribbon flat and untwisted.
- Selected connector: top-contact `DM-ACC05045-908`.

#### Selected connector mechanical data

- Manufacturer named on the provided drawing: Shenzhen Holy Electronic Co., Ltd.
- Drawing/series: `FPC0520SQ-nP`.
- Type: 0.5 mm-pitch, 2.0 mm-high SMT ZIF, marked `UPPER CONN` (top contact).
- The drawing includes recommended FPC/FFC dimensions and a recommended PCB land pattern.
- With `n = 45`, the contact span is `22.00 mm`, and the related layout span is `23.00 mm`.
- Recommended mating FPC thickness: `0.30 +0.03/−0.02 mm` (0.28–0.33 mm).
- Ratings: 50 V AC/DC, 0.5 A AC/DC, contact resistance ≤20 mΩ, insulation resistance ≥100 mΩ as printed, and operating temperature −25 °C to +85 °C.
- Source for all connector data above: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.
- The connector nominal thickness matches the display's 0.30 mm stiffener, but the connector minimum is 0.28 mm while the display tolerance extends to 0.27 mm. Retain DisplayModule's recommendation as compatibility evidence or request explicit full-tolerance confirmation.
- Before final footprint approval, visually identify terminal 1 on the recommended PCB layout, map it to KiCad pad 1, and compare a 1:1 footprint printout with the connector/FPC.

#### Verified FPC mechanical requirements

- 45 FPC positions; pin 1 is `XR` and pin 45 is `LEDA`.
- The module mechanical drawing marks pin 1, pin 45, and the FPC contact side.
- Contact pitch: `0.5 mm` (`P0.5 × 44 = 22.00 ± 0.05 mm`).
- Tail width: `23.00 ± 0.10 mm`.
- Stiffened insertion-area thickness: `0.30 ± 0.03 mm`.
- Base FPC thickness: `0.10 ± 0.03 mm`.
- The display module and connector drawings now verify the mating pitch, nominal thickness, contact orientation, and recommended PCB land pattern. Terminal-1 mapping and the 0.01 mm minimum-tolerance difference remain final approval checks.

#### Connector-verification references

- Pin functions and numbering: `hollix_data_sheets/display_datasheet.pdf`, PDF p. 4.
- FPC pin-end markings, contact side, pitch, width, and thicknesses: `hollix_data_sheets/display_datasheet.pdf`, PDF p. 5.
- Connector manufacturer, type, dimensions, mating-FPC requirement, and recommended PCB land pattern: `hollix_data_sheets/display_mechanical_datasheet.pdf`, PDF p. 1.
- Controller-document scope: `hollix_data_sheets/DM-TFT40-388_20Controller_20Datasheet.pdf`, PDF p. 1. This is the ST7701S IC/controller datasheet and is not the source for panel-connector mechanics.

#### Intended mechanical and optical arrangement

- Mount the interposer using insulated spacers so it cannot contact the LCD metal backing.
- Add FPC strain relief and keep the interposer and wiring outside the optical area within the top enclosure.
- The angled reflective arrangement may horizontally mirror the viewed image; verify with a directional test pattern and mirror the framebuffer or artwork if required.
- Prototype viewing angle, brightness, reflection alignment, and enclosure clearances before finalizing the enclosure.

#### Data Sheet: [[hollix_data_sheets/display_datasheet.pdf]]
#### Controller DataSheet: [[hollix_data_sheets/DM-TFT40-388_20Controller_20Datasheet.pdf]]
#### Connector Mechanical Data Sheet: [[hollix_data_sheets/display_mechanical_datasheet.pdf]]

#### Initialization Code


### MCU - Target Prototype: NUCLEO-144 STM32H753ZI EVAL BRD

Current development board and transition target: NUCLEO-H753ZI (on hand)

Previous target MCU: STM32H750XBH6

Previous board, discontinued: NUCLEO-H743ZI2
- Data sheets and information on the stm32 Website 

#### Data Sheet: [[DS_stm32h753zi.pdf]]

#### User Manual: [[um2407-stm32h7-nucleo144-board-stmicroelectronics.pdf]] 

### Modern high-voltage backlight driver IC - Purchased from Digikey - TPS61176EVM-566

#### Data Sheet: [[dataSheet_tps61176.pdf]]

#### User Manual: [[User_guide_slvu998.pdf]]

## Battery Charging / Power Path Control : MCP73871 Click

#### Data Sheet: [[MCP73871-Data-Sheet-DS20002090F.pdf]]

#### Schematic: [[mcp73871-click-schematic-v100.pdf]]

#### Source of Purchase: https://www.mikroe.com/mcp73871-click?utm_source=chatgpt.com



## Voltage Regulator - Buck Regulator - SparkFun AP63203 3.3

#### Data Sheet: [[DataSheet_buckRegulator_AP63200-AP63201-AP63203-AP63205.pdf]]

#### User Information: https://learn.sparkfun.com/tutorials/buck-regulator-hookup-guide?utm_source=chatgpt.com

#### Github: https://github.com/sparkfun/Buck_Regulator_AP63203?utm_source=chatgpt.com

 - NEEDS Capacitors


## Power - LiPo 4000 mAh 1S pack

https://meshnology.com/products/3-7v-4000mah-lipo-battery-with-usb-charger-for-arduino-esp32-drone?variant=46431264080110&utm_source=chatgpt.com

#### Datasheet/Product info: [[3.7V 4000mAh Micro JST 1.25 LiPo Rechargeable Battery Info]]

FainWan 20 Pairs JST 1.25mm 2 Pin Micro Male Female Connector Plug 80mm Wires Cables