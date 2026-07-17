# Decisions

## 2026-07-14 — Linear power tree: XT-60 battery → LM7805 → AMS1117-3.3
Battery power enters on an XT-60 connector (J8) through master switch SW4 onto `+12V`, then through
cascaded linear regulators: LM7805 to `+5V` and AMS1117-3.3 to `+3.3V`, with ferrite-filtered
`Vdrive` (5 V) and `VDDA` (3.3 V analog) branches. Linear regulation was chosen over switching for
noise simplicity at the cost of efficiency and heat in the 12 V → 5 V drop. Moving to a switcher
later means redesigning the power section and its layout.

## 2026-07-14 — microSD socket from a converted EasyEDA footprint (LCSC C164170)
The microSD socket (U8) has no KiCad-official footprint, so the LCSC part C164170 was converted with
`easyeda2kicad` into project-local libraries: symbol in `pcb/libs/custom-symbols.kicad_sym`,
footprint `easyeda:TF-SMD_472192001`, STEP/WRL models in `pcb/libs/easyeda.3dshapes/`. This pins the
design to the orderable LCSC part. Gotcha: with `--project-relative`, the tool emits a doubled
`pcb/pcb/` 3D-model path (KIPRJMOD already points at `pcb/`) that must be fixed by hand after every
re-conversion.

## 2026-07-12 — ATmega328P-AU telemetry co-processor on its own subsheet
Telemetry — the nRF24L01P 2.4 GHz link, CH340G USB-serial, and microSD logging — runs on a dedicated
ATmega328P-AU instead of loading the STM32, keeping RF and logging off the flight-control MCU. It is
captured as the hierarchical subsheet "ATMEGA238P" (`ATMEGA238P-plus-components.kicad_sch`), connected
to the root sheet through global labels and shared rails rather than sheet pins. Folding this back
into a single MCU would re-architect the schematic and both pin maps.

## 2026-07-12 — Adopt the new-project doc structure
Scaffolded root `CLAUDE.md` plus `agents/{CONTEXT,roadmap,decisions}.md` so the repo matches the
`/new-project` skill's canonical layout and a future run refines rather than clobbers. The Codex
`AGENTS.md` is kept alongside it — the two conventions coexist and are cross-linked, not merged.

## 2026-07-11 — KiCad 10 file formats (not backward-compatible)
The design is saved in KiCad 10 formats (schematic `20260306`, PCB `20260206`). KiCad 9 reports
"Failed to load" for both files, so KiCad 10+ is mandatory for every contributor and for CI/CLI checks.
Reverting to KiCad 9 would require re-authoring the design; treated as one-way.

## 2026-07-11 — STM32F103R8T6 (LQFP-64) as the MCU
The flight-controller core is fixed to the STM32F103R8T6 in LQFP-64. The pin map, power topology, and
footprint choices downstream all depend on this part, so changing it is a re-layout.

## 2026-07-11 — Two-layer PCB stackup
The board targets a nominal two-layer stackup. This constrains ground-plane strategy, routing density,
and impedance control; moving to four layers later would be a significant re-spin.

## 2026-07-11 — BOOT0 defaults to user-Flash boot
BOOT0 will be pulled to its user-Flash-boot state by default, with any system-memory (ROM bootloader)
entry exposed as a deliberate, controlled override rather than the resting state.

## 2026-07-11 — Codex AGENTS.md only; no .codex/config.toml
Repository agent instructions live in `AGENTS.md`. A project `.codex/config.toml` is intentionally
omitted because no repo-specific model, sandbox, MCP, or hook setting is required.
