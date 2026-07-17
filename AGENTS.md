# Repository instructions

## Scope

This repository contains a KiCad 10 hardware design for a drone flight controller. The editable design lives in `pcb/`:

- `drone-flight-controller.kicad_pro` — project settings and design rules
- `drone-flight-controller.kicad_sch` — source schematic, root sheet (STM32 core and power tree)
- `ATMEGA238P-plus-components.kicad_sch` — source schematic, telemetry subsheet (ATmega328P, nRF24L01P, CH340G, microSD)
- `drone-flight-controller.kicad_pcb` — source PCB layout (currently empty)
- `libs/`, `sym-lib-table`, `fp-lib-table` — project-local libraries (custom microSD symbol, EasyEDA footprint and 3D model)
- `datasheets/` — local manufacturer documentation used as review evidence

Treat `.kicad_sch`, `.kicad_pcb`, and `.kicad_pro` as source files. Do not edit generated backups or `.kicad_prl` unless the task specifically requires it.

## Toolchain

- Use KiCad 10 or newer. The files use KiCad 10 formats (`20260306` schematic and `20260206` PCB) and KiCad 9 cannot load them.
- Prefer `kicad-cli` for non-interactive ERC, DRC, plotting, and exports.
- Use deterministic KiCad parsers or `pcbnew` for structural inspection; do not rewrite S-expressions with broad text substitutions.
- Do not assume this directory is a functioning Git worktree. Check before using Git-based workflows.

## Required workflow

Before editing, inspect `README.md` and `agents/DESIGN_REVIEW.md` and identify whether the task affects the schematic, PCB, or both.

For schematic changes:

1. Confirm the component pinout and required application circuit against a manufacturer datasheet.
2. Add manufacturer part numbers and footprints before calling a component fabrication-ready.
3. Run ERC with KiCad 10:
   `kicad-cli sch erc pcb/drone-flight-controller.kicad_sch --output /tmp/drone-flight-controller-erc.rpt`
4. Review every new error and warning; do not suppress findings without documenting why.

For PCB changes:

1. Update the PCB from the schematic before placement or routing.
2. Refill zones after copper edits.
3. Run DRC with KiCad 10:
   `kicad-cli pcb drc pcb/drone-flight-controller.kicad_pcb --output /tmp/drone-flight-controller-drc.rpt`
4. Confirm board outline, unrouted count, ground-plane continuity, decoupling placement, and footprint pad numbering.

For changes touching both domains, cross-check every schematic pin/net against its PCB pad/net assignment. A clean ERC/DRC does not replace datasheet pinout verification.

## Design constraints and review priorities

- The design is a complete two-sheet schematic — STM32F103R8T6 flight-controller core with power tree, plus an ATmega328P telemetry subsheet — but the PCB is empty and open items remain; see `agents/DESIGN_REVIEW.md` before claiming any milestone.
- `BOOT0` is strapped through `R3` to the boot-select jumper `J2` (+3.3 V / GND). Keep user-Flash boot as the default strap unless requirements say otherwise.
- Preserve one 100 nF bypass capacitor per VDD pin and the 4.7 uF bulk capacitor placement specified for VDD3.
- Keep VDDA/VSSA tied to the digital supply domains through an intentional, documented filter. `L1` still carries the invalid value `27nF`; its component type and value remain unresolved.
- Give external connectors ESD protection and explicit ground-return paths.
- Keep high-current motor/ESC power paths separate from sensitive IMU/ADC supplies and provide a continuous ground reference.
- Do not claim fabrication readiness without completed ERC, DRC, BOM/MPN coverage, Gerber review, and schematic-to-PCB synchronization.

## Generated and local files

- Do not modify files in `pcb/drone-flight-controller-backups/`.
- Keep temporary ERC/DRC reports, analyzer JSON, plots, and fabrication outputs out of the source tree unless the user asks to commit a review artifact.
- Preserve the local STM32 datasheet. Cite its page, figure, or table when it is used as evidence.

## Handoff

Summarize changed source files, electrical or layout assumptions, validation commands actually run, tool versions, unresolved findings, and any checks skipped because KiCad 10 or other required tooling was unavailable.
