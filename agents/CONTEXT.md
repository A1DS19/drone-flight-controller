# Drone Flight Controller — Context

Canonical vocabulary for a custom STM32F103-based drone flight-controller PCB. Domain terms only; see
`decisions.md` for choices and `roadmap.md` for scope.

## Board & parts

**Flight controller**:
The board being designed — the electronics that read sensors and drive the motor ESCs to stabilise a
drone. Schematic capture is complete (two hierarchical sheets); the PCB layout has not been started.

**MCU**:
The STM32F103R8T6 (LQFP-64), `U1` — the flight-control core on the root sheet.
_Avoid_: processor, CPU.

**Telemetry MCU**:
The ATmega328P-AU (TQFP-32), `U3` — a co-processor on the subsheet that owns the radio link, USB-serial,
and microSD logging. Note: the subsheet and its filename spell it "ATMEGA238P"; the part is a 328P.

**Radio**:
The nRF24L01P 2.4 GHz transceiver (`U7`) with coaxial antenna connector `J17` and an L-network match
(`L2`–`L4`).

**microSD socket**:
`U8`, LCSC C164170 — project-local symbol `custom-symbols:MICROSD_C164170`, footprint
`easyeda:TF-SMD_472192001`. Wired for SPI-mode access with a grounded shell; pin electrical types are
set per function (VDD/VSS `power_in`, DAT0 `output`, CMD/DAT `bidirectional`, shell tabs `passive`).

**Level shifting**:
74HC245 (`U6`) and 74AHC1G125 (`U9`) moving the SPI bus between the 5 V and 3.3 V domains.

**Decoupling network** (a.k.a. bypass network):
The 100 nF capacitors placed one per VDD/VSS pin pair plus the 4.7 uF bulk capacitor on VDD3, per the
STM32 datasheet Figure 14. Their job is local charge delivery and supply-noise suppression.
_Avoid_: filter caps.

**Bulk capacitor**:
The single 4.7 uF device on the digital supply that backs the per-pin 100 nF caps.

**VDDA filter**:
The L–C network feeding the analog supply (VDDA) from the digital rail to keep ADC/PLL noise down.
Inductor `L1` still carries an invalid `27nF` value and no footprint — its electrical intent is
unresolved.

**BOOT0**:
The STM32 pin that selects the startup boot source (user Flash vs system-memory bootloader). No longer
floating: strapped through `R3` (10 kΩ) to boot-select jumper `J2` (+3.3 V / GND). User-Flash boot is
the intended default strap position.

**NRST**:
The MCU reset pin. Has a permanent internal 30–50 kΩ pull-up; externally it also gets `R1` (10 kΩ) to
+3.3 V, `C9` (100 nF), and reset button `SW1` on net `RST`.

## Rails & nets

**Rail**:
A named supply net. The power tree is: `+12V` (XT-60 battery input `J8`) → `SW4` master switch →
`+12C` (switched 12 V; "C" as in *conmutado*) → `FB1` → LM7805 → `+5V` → AMS1117-3.3 → `+3.3V`.
Derived rails: `Vdrive` (ferrite-filtered 5 V branch) and `VDDA` (filtered analog 3.3 V). Plus `GND`.
No `PWR_FLAG` symbols exist yet, so ERC will report undriven power inputs on these rails.

**Net-name quirks**:
Labels are partly Spanish — `BOTON` (user button), `MANDO_1..6` (RC-receiver channels), LED values
`ROJO`/`VERDE`/`AZUL`. The I²C buses are labelled `12C1_*` and `12C2_*` with a digit "12", not "I2" —
searches for "I2C" will miss them.

**Net class**:
A KiCad group of nets sharing clearance/width/via rules. Only the `Default` class is defined
(0.20 mm clearance, 0.20 mm track, 0.60/0.30 mm via).

## Tooling & review

**ERC** (Electrical Rules Check):
KiCad's schematic-level rule check. Run with `kicad-cli sch erc` — the CLI must come from KiCad 10;
the 9.x CLI fails with "Failed to load". Native ERC has never been run on this design.

**DRC** (Design Rules Check):
KiCad's PCB-level rule check. Run with `kicad-cli pcb drc` (same KiCad 10 requirement). Not meaningful
yet — the board is empty.

**MPN** (Manufacturer Part Number):
The orderable part identifier attached to a component. Coverage is one part: `D3` (SRV05-4A,
LCSC C20615829). DigiKey-sourced symbols (`U3`, `U5`) carry MPN-like values but no explicit MPN field.

**Footprint**:
The physical land pattern a schematic symbol maps to on the PCB. Assigned only on the ten major ICs
(`U1`–`U5`, `U7`, `U8`, `U10`, `U11`, `D3`); every passive, connector, crystal, switch, and LED —
roughly 148 of 158 components — still has none.

**Source files**:
The hand-editable KiCad files: `.kicad_pro`, the two `.kicad_sch` sheets, `.kicad_pcb`, plus the
project libraries (`pcb/libs/`, `sym-lib-table`, `fp-lib-table`). Everything else (`.kicad_prl`,
backups, lock files, exports) is local or generated.
