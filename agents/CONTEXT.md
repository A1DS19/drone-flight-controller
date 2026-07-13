# Drone Flight Controller — Context

Canonical vocabulary for a custom STM32F103-based drone flight-controller PCB. Domain terms only; see
`decisions.md` for choices and `roadmap.md` for scope.

## Board & parts

**Flight controller**:
The board being designed — the electronics that read sensors and drive the motor ESCs to stabilise a
drone. Today it is only an MCU power fragment, not yet a functional flight controller.

**MCU**:
The STM32F103R8T6 (LQFP-64) microcontroller at the centre of the design.
_Avoid_: processor, CPU.

**Decoupling network** (a.k.a. bypass network):
The 100 nF capacitors placed one per VDD/VSS pin pair plus the 4.7 uF bulk capacitor on VDD3, per the
STM32 datasheet Figure 14. Their job is local charge delivery and supply-noise suppression.
_Avoid_: filter caps.

**Bulk capacitor**:
The single 4.7 uF device on the digital supply that backs the per-pin 100 nF caps.

**VDDA filter**:
The L–C network feeding the analog supply (VDDA) from the digital rail to keep ADC/PLL noise down.
Inductor `L1` currently carries an invalid `27nF` value and no footprint — its electrical intent is
unresolved.

**BOOT0**:
The STM32 pin that selects the startup boot source (user Flash vs system-memory bootloader). Must have
a defined default state; currently floating.

**NRST**:
The MCU reset pin. Has a permanent internal 30–50 kΩ pull-up, so an external pull-up is not required.

## Rails & nets

**Rail**:
A named supply net. Only three exist today: `+3.3V` (digital), `VDDA` (analog), and `GND`. Both `+3.3V`
and `VDDA` are consumer-only — no regulator or input source drives them yet.

**Net class**:
A KiCad group of nets sharing clearance/width/via rules. Only the `Default` class is defined
(0.20 mm clearance, 0.20 mm track, 0.60/0.30 mm via).

## Tooling & review

**ERC** (Electrical Rules Check):
KiCad's schematic-level rule check. Run with `kicad-cli sch erc`.

**DRC** (Design Rules Check):
KiCad's PCB-level rule check. Run with `kicad-cli pcb drc`.

**MPN** (Manufacturer Part Number):
The orderable part identifier attached to a component. MPN coverage is currently 0% — no component is
fabrication-ready.

**Footprint**:
The physical land pattern a schematic symbol maps to on the PCB. Nine of ten components have none.

**Source files**:
The three hand-editable KiCad files (`.kicad_pro`, `.kicad_sch`, `.kicad_pcb`). Everything else
(`.kicad_prl`, backups, lock files, exports) is local or generated.
