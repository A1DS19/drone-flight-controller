# Drone Flight Controller Design Review

**Date:** 2026-07-19
**Design format:** KiCad 10, two-sheet hierarchical schematic, nominal two-layer PCB
**Verdict:** Schematic capture is complete, the first native ERC ran clean (0/0, 2026-07-19), and
footprints are assigned to every component and imported into the board. Layout (outline, placement,
routing) can begin. Fabrication is still blocked by LCSC/MPN coverage (4 parts), `L1`, and DRC.

## Critical findings

| Severity | Finding | Evidence |
|---|---|---|
| Blocker | The PCB has all 158 footprints imported (2026-07-19) but no outline, placement, routing, or zones yet. | `drone-flight-controller.kicad_pcb` |
| Blocker | LCSC/MPN coverage is four parts (`D3` C20615829, `SW1`–`SW3` C7528713) — the BOM is not orderable. Footprints, by contrast, are now assigned to all ~158 components (2026-07-19). `SW4`, the master power switch, is deliberately still unsourced (a momentary tact part is unsuitable); it carries a placeholder `easyeda:Switch_Slide` footprint. | Schematic property scan, netlist import log |
| Resolved | The USB shield gap from the Update-PCB import is fixed (2026-07-19): J3/J16's shield pin was renumbered "SH" → "6" to match the footprint's shell pads and tied to GND (`#PWR0142`/`#PWR0143`). Re-run Update PCB to carry it onto the board. `U7`'s exposed pad (footprint pad 21, no symbol pin) is **intentionally unconnected**: the nRF24L01+ Product Specification v1.0, page 65, recommends keeping the die attach pad unconnected — the recurring import warning for pad 21 is expected and benign. (The former SW1–SW3 pad-3/4 warnings disappeared with the TC-6610 swap: the official footprint's paired 1,1,2,2 pads all map.) | Netlist import log, nRF24L01+ PS v1.0 p. 65 |
| Resolved | Headless KiCad 10 checks are available after all: the AppImage bundles `kicad-cli` 10.0.4 at `/home/dev/Applications/kicad-10/AppDir/bin/kicad-cli` (discovered 2026-07-19; ERC verified working). The system `kicad-cli` 9.0.9 still cannot load these files — use the AppImage path. | `kicad-cli version` → 10.0.4 |
| Warning | `L1` in the VDDA filter still carries the invalid value `27nF` and has no footprint or MPN. The filter's electrical intent remains unresolved. | Root schematic, `Device:L_Small` |
| Resolved | First native ERC ran clean on 2026-07-19 (0 errors / 0 warnings, KiCad 10 GUI) after: `PWR_FLAG`s on the passive-fed rails (`+12V`, `+12C`, `VDDA`, the FB1→LM7805 node, `Vdrive`) and one on `GND`; a no-connect on `SW4`'s unused throw; the USB shell GND pins retyped to passive; and a real bug fix — `U4` (CH340G) had VCC and V3 cross-wired, leaving the chip unpowered with its internal 3.3 V node tied to 5 V. | `pcb/ERC.rpt` 2026-07-19T16:54 |
| Warning | The telemetry subsheet has no sheet pins; all cross-sheet connectivity rides on global labels and power symbols. Functional, but every net is effectively global, and the I²C buses are named `12C1_*`/`12C2_*` (digit "12"), which text searches for "I2C" will miss. | Sheet definition + label scan |
| Warning | `pcb/sym-lib-table` is empty — the `custom-symbols` and DigiKey/PCM libraries are not registered project-locally and resolve only through the embedded symbol cache and the user's global table. Both lib tables are untracked in git. | `pcb/sym-lib-table`, `git status` |

## What is present

Roughly 158 components across 82 named nets on two sheets.

**Sheet 1 — flight controller core** (`drone-flight-controller.kicad_sch`, 88 components):

| Subsystem | Parts |
|---|---|
| MCU | `U1` STM32F103R8T6, LQFP-64, footprint assigned |
| IMU | `U2` MPU-6050 (QFN-24) on I²C bus `12C2_*` |
| Clocks | `Y1` 8 MHz HSE (20 pF loads), `Y2` 32.768 kHz LSE (10 pF loads) |
| Power | `J8` XT-60 → `SW4` master switch → `U10` LM7805 (TO-220) → `U11` AMS1117-3.3 (SOT-223); `FB1`/`FB2` ferrites, `D1`/`D2` B5819W Schottkys, `R17`/`R18` 47k/15k battery divider → `ADC_Vbat` |
| Reset/boot | `SW1` (TC-6610, LCSC C7528713) + `R1` 10k + `C9` 100 nF on NRST; `J2` boot jumper + `R3` 10k on BOOT0 |
| USB | `J3` micro-B with `D3` SRV05-4A ESD array (LCSC C20615829) |
| I/O | `J1` SWD, `J4`–`J7` ESC/motor, `J9` TF-Luna LIDAR, `J10` RC receiver (`MANDO_1..6`), `J11` OLED, `J12`–`J14` SPI/I²C/USART expansion |
| Misc | Status LEDs, `SW2` user button (`BOTON`, TC-6610), `H1`–`H4` mounting pads on `+12V`, `H5`–`H8` on `GND` |

**Sheet 2 — telemetry** (`ATMEGA238P-plus-components.kicad_sch`, hierarchical subsheet, 70 components):

| Subsystem | Parts |
|---|---|
| MCU | `U3` ATMEGA328P-AU (TQFP-32), `Y3` 16 MHz, `J15` ICSP, `SW3` reset (TC-6610) |
| Radio | `U7` nRF24L01P (QFN-20), `J17` coaxial antenna, `L2`–`L4` matching, `Y5` 16 MHz |
| USB-serial | `U4` CH340G (SOIC-16), `J16` micro-B, `Y4` 12 MHz |
| Level shift | `U6` 74HC245, `U9` 74AHC1G125 (5 V ↔ 3.3 V SPI) |
| Storage | `U8` microSD socket (LCSC C164170), SPI-mode wired: CS=`CS_SD_74HC245`, MOSI/CLK from the 74HC245, MISO=`MISO_SD_OUT` via `U9`, VDD=+3.3V, VSS+shell=GND, DAT1/DAT2 no-connected |
| Analog | `U5` LM358 dual op-amp on the RX/TX path |

## Power network

```text
J8 XT-60 (+12V) --- SW4 --- (+12C) --- FB1 --- U10 LM7805 --- (+5V) --- U11 AMS1117-3.3 --- (+3.3V)
      |                                                          |                             |
      R17/R18 divider -> ADC_Vbat                                FB2 -> Vdrive (5 V)           L1 (?) -> VDDA
                                                                                                +-- C7: 1 uF
GND ---------------------------------------------------------------------------------------- +-- C8: 100 nF
```

- The STM32 digital decoupling matches the datasheet: Figure 14 on page 36 calls for five 100 nF
  capacitors plus one 4.7 uF on VDD3, and the schematic has the corresponding quantities.
- For VDDA the same figure recommends 10 nF + 1 uF; the schematic uses 100 nF + 1 uF after `L1`. This
  may work but remains an unjustified deviation, and `L1` itself is unresolved (see findings).
- `+12C` is the switched 12 V node between `SW4` and `FB1` — intentional naming, not a stray net.
- `PWR_FLAG`s were added 2026-07-19 on the passive-fed rails (`+12V`, `+12C`, `VDDA`, FB1→LM7805,
  `Vdrive`) and once on `GND`. Two GND symbol libraries remain mixed (`power:GND` and
  `PCM_SparkFun-PowerSymbol:GND`), which is cosmetic only.

## Reset and boot

Both prior blockers are resolved in the schematic:

- NRST: internal 30–50 kΩ pull-up (datasheet Table 39, page 66) plus external `R1` 10 kΩ to +3.3 V,
  `C9` 100 nF (matching the recommended Figure 31, page 67 protection), and reset button `SW1`.
- BOOT0: `U1` pin 60 runs through `R3` 10 kΩ to the center pin of boot-select jumper `J2`, whose outer
  pins sit on +3.3 V and GND. Per `decisions.md`, the default strap must be the GND (user-Flash)
  position — confirm the shipped jumper orientation at assembly.

## PCB and EMC

All 158 footprints are imported and the 95 mm frame outline template is placed; placement and
routing have not started, so no EMC review yet. Once placement begins, use
a continuous ground plane, place each bypass capacitor adjacent to its VDD/VSS pair, keep the VDDA
filter local to the MCU, keep the nRF24 antenna matching and feed clear of the power section, and keep
motor/ESC return currents away from the IMU and `ADC_Vbat` sensing.

## Project rules and compatibility

- Only the default net class exists: 0.20 mm clearance, 0.20 mm track, 0.60/0.30 mm via, 0.50 mm
  copper-to-edge. No power, RF, analog, or clock classes yet.
- Files are KiCad 10 (schematic `20260306`, PCB `20260206`). The system `kicad-cli` 9.0.9 cannot
  load them; use the AppImage CLI (`/home/dev/Applications/kicad-10/AppDir/bin/kicad-cli`, 10.0.4).
- `pcb/fp-lib-table` registers the `easyeda` footprint library; `pcb/sym-lib-table` is empty (see
  findings). Both lib tables are tracked in git.

## Analyses performed

- Structural s-expression inspection of both schematic sheets: performed
- Net connectivity extraction (KiCad parser): performed — 82 named nets
- Pin-geometry verification of `U8` attachments (wires, labels, no-connects, power symbols): performed
- Footprint/MPN property scan across all components: performed
- Native ERC (KiCad 10 GUI): performed 2026-07-19 — clean after the seven initial errors were fixed
- Update-PCB-from-schematic import of all 158 footprints: performed 2026-07-19 (2 errors / 19
  warnings logged; see findings)
- STM32 datasheet cross-check of power, reset, and boot networks: performed (carried forward and
  re-verified against the current schematic)

## Not performed / review limits

- Native DRC: not yet run (board has no layout); headless ERC/DRC remain blocked by the 9.0.9 CLI.
- Pin-level datasheet verification of the telemetry subsheet (ATmega328P, nRF24L01P, CH340G, 74HC245,
  74AHC1G125, microSD): not performed — only the STM32 power/reset/boot network is formally checked.
- SPICE, Gerber/DFM, lifecycle/sourcing: not applicable yet (no layout; MPN coverage is four parts).

## Recommended next sequence

1. Re-run Update PCB from Schematic to carry the USB-shield fix onto the board, and re-run ERC
   (expected to stay clean; the two new GND symbols sit on the already-driven GND net).
2. Resolve `L1` (real inductor or ferrite value; it now carries an L_0402 footprint but the `27nF`
   value is still invalid).
3. Register the project symbol libraries in `pcb/sym-lib-table` and note the board's new external
   footprint-library dependencies (`digikey-footprints`, `PCM_JLCPCB`) resolve only through the
   global table.
4. Assign MPNs/LCSC to the remaining ~154 parts, including a latching, adequately rated part for
   the `SW4` master switch.
5. Define outline, stackup, net classes, ground plane; place and route.
6. Run KiCad 10 DRC, cross-domain/EMC review, BOM/MPN review, and Gerber/DFM checks before fabrication.
