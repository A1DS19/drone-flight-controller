# Session handoff — 2026-07-20

## What was done (this session, main @ 94f3ab2)
- First clean native ERC (0/0) — fixed en route: CH340G VCC/V3 cross-wiring (real bug), PWR_FLAGs
  on all passive-fed rails + GND, SW4 no-connect, USB shell pin types.
- Footprints assigned to all ~158 parts and imported to the board; 95 mm frame outline placed.
- Every part sourced: switches (TC-6610), USB (MicroXNJ), SW4 (SS12D10G5), L1 (BLM15 bead +
  datasheet-exact VDDA caps), full jellybean sweep (142 refs, 61 groups). BOM gate: PASS,
  0 blocking, 26 Extended (~$78 max setup). See pcb/parts-log.md.
- 3D models complete (official footprints for U3/U5/U10; converted assets for USB/switches/SD).
- USB shield fix made structural after two symbol-update regressions: corrected USB_B_Micro
  lives in pcb/libs/custom-symbols.kicad_sym, registered in sym-lib-table, J3/J16 point at it.
- Last netlist import: 0 errors, 1 benign warning (U7 exposed pad — unconnected per nRF24L01+
  datasheet p. 65; ignore it forever).

## Current state
Schematic: done (electrically complete, sourced, clean ERC). Board: all footprints + frame
outline, no placement/routing/zones yet. User is doing placement and routing by hand.

## Plan / how to help next session
- User does placement + routing. Support roles: placement review (pcb-placement-review skill),
  DRC runs via the AppImage CLI (`/home/dev/Applications/kicad-10/AppDir/bin/kicad-cli pcb drc`),
  net classes / design rules setup, Gerber + BOM/CPL export when ready.
- Placement constraints (DESIGN_REVIEW "PCB and EMC"): continuous GND plane; each 100 nF at its
  VDD/VSS pair; VDDA filter (L1/C7/C8) tight to U1 pin 13; nRF24 matching + antenna feed clear of
  power; ESC/motor returns away from IMU and ADC_Vbat divider; LM7805 tab bolted over GND copper.
- Gotchas that bit before: never "Update Symbols from Library" casually; F8 needs the
  replace-footprints checkbox only after footprint changes; close KiCad before I edit files;
  commit before risky GUI operations.
