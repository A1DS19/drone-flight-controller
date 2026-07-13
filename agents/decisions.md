# Decisions

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
