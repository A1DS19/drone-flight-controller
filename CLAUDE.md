# Drone Flight Controller

A custom STM32F103R8T6-based drone flight controller, designed in KiCad 10. Schematic capture is
essentially complete across a two-page hierarchical schematic: sheet 1 holds the STM32 core (MPU-6050
IMU, ESC/receiver/LIDAR I/O, SWD debug, USB with ESD protection) and the XT-60 → LM7805 → AMS1117-3.3
power tree; sheet 2 ("ATMEGA238P") holds an ATmega328P telemetry co-processor with an nRF24L01P radio,
CH340G USB-serial, level shifting, and a microSD socket. The PCB layout has not been started, and open
items remain before it can — footprint/MPN coverage stops at the major ICs, the custom microSD
symbol's pin types are wrong, and native ERC has never run (see
[agents/DESIGN_REVIEW.md](agents/DESIGN_REVIEW.md)). It is a hardware
design, not software — there is no build; "test" means running KiCad's electrical and design rule
checks.

## Stack
- **KiCad 10** (schematic format `20260306`, PCB format `20260206`) — KiCad 9 cannot load these files.
- **`kicad-cli`** for non-interactive ERC/DRC and exports.
- Main MCU: **STM32F103R8T6**, LQFP-64. Telemetry MCU: **ATmega328P-AU**, TQFP-32. Nominal **two-layer** PCB.
- Source of truth lives in `pcb/`: `.kicad_pro` (settings), `.kicad_sch` (schematic — root sheet plus
  the `ATMEGA238P-plus-components.kicad_sch` subsheet), `.kicad_pcb` (layout), `libs/` and the two
  lib tables (project libraries).

## Run & test
Open the project in KiCad 10+: `pcb/drone-flight-controller.kicad_pro`.

Electrical and design rule checks (the closest thing to a test suite):

```sh
kicad-cli sch erc pcb/drone-flight-controller.kicad_sch --output /tmp/drone-flight-controller-erc.rpt
kicad-cli pcb drc pcb/drone-flight-controller.kicad_pcb --output /tmp/drone-flight-controller-drc.rpt
```

A clean ERC/DRC is necessary but not sufficient — verify symbol/footprint pin numbering against the
manufacturer datasheet and review Gerbers before ordering.

## Conventions
- **Hardware workflow, edit safety, and review priorities are in [AGENTS.md](AGENTS.md)** — read it
  before editing the schematic or PCB. It defines what counts as a source file, the required ERC/DRC
  steps, and the power/BOOT0/ground constraints.
- **Current engineering state and blockers are in [agents/DESIGN_REVIEW.md](agents/DESIGN_REVIEW.md).**
- Domain vocabulary → [agents/CONTEXT.md](agents/CONTEXT.md). Hard, costly-to-reverse decisions →
  [agents/decisions.md](agents/decisions.md). Scope and milestones → [agents/roadmap.md](agents/roadmap.md).
- Treat `.kicad_sch`, `.kicad_pcb`, and `.kicad_pro` as source. Do not edit backups or `.kicad_prl`, and
  keep ERC/DRC reports, plots, and fabrication outputs out of the source tree (they are `.gitignore`d).
- Edit KiCad files with KiCad tooling/parsers, not broad S-expression text substitutions.
- This is not always a Git worktree — check before relying on Git workflows.

## Session handoff
- At session start, read `agents/handoff.md` first if it exists.
- When the user ends a session with "let's continue tomorrow" (or similar), overwrite `agents/handoff.md`
  per its format: what was done, current state, and the plan for next session.
