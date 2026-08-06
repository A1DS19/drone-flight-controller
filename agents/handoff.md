# Session handoff — 2026-07-30

## What was done (this session)
- **Crystal-placement review** (user's request, all 5 crystals): the AN2867 Fig-14 moated
  In1 islands are sound — tight loops (caps 1.4–2.8 mm, MCU pins 3.3–6.3 mm), full island
  coverage under every crystal, guard rings + stitching on Y3/Y5, 0 blocking. Found the
  STM32 islands' single-point bridges sat AWAY from the trace-entry corridor: all four
  HSE/LSE nets crossed the moat over voids (ZONE-REF worst 0% reference).
- **Fixed the STM32 bridges** (AppImage pcbnew, not MCP): two micro-bridges replaced by one
  L-shaped `GND bridge osc stm32` at (122.5,97.2)–(126.5,100.4), top edge stepped to
  y=97.45 for x>124.2 to clear the +3.3V via at (124.9,96.6). Islands now merge through
  one neck under U1 pins 3–6. Zones gate 21→17 warnings / 0 blocking; DRC violation
  histogram byte-identical pre/post (401, all pre-existing).
- **Skills** (claude-configs branch `feat/osc-bridge-at-entry`, commit 0df4ab8, NOT pushed):
  CHECKLIST 5.8 (bridge under trace entry) + digest line 6, design-rules 16b extension,
  zones.py `osc_nets` model key + `[oscillator net]` ZONE-REF annotation with the
  bridge-at-entry recommendation. Suites 85/3/97 green (+1 new test).

## Current state
- Drone tree UNCOMMITTED: user's zone/crystal/routing WIP plus today's bridge fix in
  `.kicad_pcb`; `.kicad_pro` modified; crystal datasheets (AN2867, SPMA056) untracked.
- Osc clusters are locally routed on F.Cu (the 07-27 "1 segment" note is stale).
- Open board items: (1) NRF_XC1 still 50% covered — Y5's bridge is at the entry but
  narrow, user scoped today's fix to STM32 only; (2) NEW: `+3.3V` In2 zone splits into
  2 disconnected regions — resolve before routing; (3) `+12C` PCB-only net (stale sync,
  from 07-27); (4) H1-H8/J8/J17 pads → SOLID; (5) sliver island-removal limits ~2 mm²
  (GND/In1 ×3, +5V/In2 ×6); (6) Y4 is 2.1 mm from board edge (noted, course design);
  (7) two different zones both named "GND bridge osc 1 atm328p" (nRF + CH340G bridges) —
  rename for auditability; (8) board 102×49 mm under JLCPCB 70×70 Standard-assembly floor.
- Standing gotchas: KiCAD-MCP-Server process runs in background — MCP edits corrupt the
  designator cache AND a stale MCP/GUI save can wipe work; close KiCad + avoid MCP for
  file edits; commit before risky GUI ops. Pre-edit board backup in session scratchpad.

## Plan for next session
1. Commit the drone WIP on a feature branch (zones + crystal placement + bridge fix).
2. Push `feat/osc-bridge-at-entry` in claude-configs and open the PR (user to confirm).
3. Fix open items 1–5 (+7 rename), re-run `pcb-verify/scripts/zones.py` (AppImage recipe
   in memory) — expect NRF_XC1 and the sliver/thermal warnings to clear.
4. Then routing per the 07-27 plan: signals on F.Cu over solid In1 GND; B.Cu short jogs
   only, never USB D± across In2 splits; via fanout → `gnd_vias.py` stitch → refill → DRC.
