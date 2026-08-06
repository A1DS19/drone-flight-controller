# Drone Flight Controller

KiCad 10 project for a custom STM32F103R8T6-based drone flight controller with an ATmega328P
telemetry co-processor.

| Front | Back |
|-------|------|
| ![Board front](screenshots/pcb-front.png) | ![Board back](screenshots/pcb-back.png) |
| OLED, XT60 battery connector, and SMA antenna jack fitted | LM7805 regulator and logo on the back silkscreen |

## Current state

Schematic capture is complete across a two-page hierarchical schematic, the four-layer PCB is
placed and routed, electrical and design rule checks pass with zero errors, and JLCPCB fabrication
outputs are generated in [pcb/production/](pcb/production/).

**Sheet 1 — flight controller core** (`pcb/drone-flight-controller.kicad_sch`):

- STM32F103R8T6 (LQFP-64) with 8 MHz HSE and 32.768 kHz LSE crystals and full supply decoupling;
- MPU-6050 IMU on I²C;
- SWD debug header (J1), BOOT0 boot-mode jumper (J2), reset and user buttons;
- USB micro-B (J3) with an SRV05-4A ESD protection array;
- four ESC/motor connectors (J4–J7), RC-receiver header (J10), TF-Luna LIDAR connector (J9),
  OLED/I²C header (J11), and SPI/I²C/USART expansion headers (J12–J14);
- status LEDs and eight mounting holes.

**Power tree** (sheet 1): XT-60 battery input (J8) on `+12V` → master switch (SW4) → switched rail
`+12C` → ferrite bead FB1 → LM7805 (`+5V`) → AMS1117-3.3 (`+3.3V`), with a ferrite-filtered `Vdrive`
5 V branch, a filtered `VDDA` analog rail, B5819W Schottky diodes in the supply path, and a 47k/15k
divider feeding `ADC_Vbat` for battery-voltage sensing.

**Sheet 2 — telemetry** (`pcb/ATMEGA238P-plus-components.kicad_sch`, hierarchical subsheet
"ATMEGA238P"):

- ATmega328P-AU (TQFP-32) with 16 MHz crystal and ICSP header (J15);
- nRF24L01P 2.4 GHz transceiver with coaxial antenna connector (J17) and L-network matching;
- CH340G USB-serial bridge with its own USB micro-B (J16);
- 74HC245 and 74AHC1G125 level shifting between the 5 V and 3.3 V domains;
- LM358 dual op-amp;
- microSD card socket (U8, LCSC C164170, project-local symbol and footprint) wired for SPI-mode
  access with grounded shell.

The two sheets communicate through global labels (I²C, USB pass-through, `DO_COMU`) and shared
power rails; the subsheet has no sheet pins.

**Board** (`pcb/drone-flight-controller.kicad_pcb`): four-layer, ~102 × 49 mm rounded outline.
F.Cu carries the signal routing, In1 is a solid ground plane (star ground, with moated oscillator
islands per ST AN2867 bridged at their trace entries), In2 carries the power planes (`+3.3V`,
`+5V`, `Vdrive`, and a `+12V` motor region), and B.Cu is reserved for short jogs. Motor mounts
H1–H8, the drone-frame mounting reference (on User.Drawings), and a logo on the back silkscreen
round out the mechanicals.

**Check status** (2026-08-05):

- DRC: **0 errors, 0 unconnected items**, 91 reviewed warnings — silkscreen labels in dense
  component clusters, stitching/fence vias that touch only the internal ground plane (the board
  has no outer ground pours, so KiCad always flags these), and intentional board-vs-library
  footprint divergence from deduplicating shared mounting holes.
- ERC: **0 errors**, 19 known-benign warnings (pin-type artifacts of the ESD-array and
  level-shifter symbols, power-label net-name overlaps, two deliberate embedded-symbol edits).

### Before ordering

- Review the Gerbers in a viewer and verify symbol/footprint pin numbering against the
  manufacturer datasheets — clean ERC/DRC is necessary but not sufficient.
- Hand-tune or accept the 18 reference labels that have no rule-clean home in the dense
  RF/LED/switch clusters.
- Confirm the JLCPCB assembly tier and pricing for the 102 × 49 mm outline.

See [agents/DESIGN_REVIEW.md](agents/DESIGN_REVIEW.md) for the engineering assessment history.

## Fabrication outputs

Generated 2026-08-05 into [pcb/production/](pcb/production/):

| File | Contents |
|------|----------|
| `drone-flight-controller.zip` | Gerber package: 4 copper layers (F, In1, In2, B), mask/paste/silk both sides, board outline, PTH + NPTH drill files and maps (15 files) |
| `bom.csv` | Bill of materials, 71 line items with LCSC part numbers |
| `positions.csv` | Component placements (CPL) for 146 parts |
| `designators.csv` | Designator index |
| `netlist.ipc` | IPC-D-356 netlist for electrical test |

## Open the project

Use KiCad 10 or newer and open:

`pcb/drone-flight-controller.kicad_pro`

The project was saved in KiCad 10 formats. KiCad 9 reports “Failed to load” for both the schematic
and board.

## Validate

With a KiCad 10 `kicad-cli` (KiCad 9 fails with “Failed to load”; an AppImage's bundled CLI at
`<AppDir>/bin/kicad-cli` works headless):

```sh
kicad-cli sch erc pcb/drone-flight-controller.kicad_sch \
  --output /tmp/drone-flight-controller-erc.rpt

kicad-cli pcb drc pcb/drone-flight-controller.kicad_pcb \
  --output /tmp/drone-flight-controller-drc.rpt
```

Expected: 0 ERC errors (19 benign warnings) and 0 DRC errors / 0 unconnected items (91 reviewed
warnings, see **Check status** above).

## Repository layout

```text
.
├── CLAUDE.md                 Assistant working brief (auto-loaded by coding agents)
├── AGENTS.md                 Repository instructions for coding agents
├── README.md
├── .gitignore                KiCad locks, backups, and generated outputs
├── agents/                   Project docs
│   ├── CONTEXT.md            Domain glossary
│   ├── roadmap.md            Goals and milestones
│   ├── decisions.md          Log of hard, costly-to-reverse decisions
│   ├── handoff.md            Session-to-session handoff notes
│   └── DESIGN_REVIEW.md      Engineering assessment
└── pcb/
    ├── datasheets/           Local manufacturer references (47 PDFs)
    ├── libs/                 Project libraries (custom microSD symbol, EasyEDA footprint + 3D model)
    ├── production/           JLCPCB fabrication outputs (Gerbers, BOM, CPL, IPC netlist)
    ├── sym-lib-table         Project symbol-library table
    ├── fp-lib-table          Project footprint-library table (registers `easyeda`)
    ├── drone-flight-controller.kicad_pro
    ├── drone-flight-controller.kicad_sch          Sheet 1 — STM32 core + power
    ├── ATMEGA238P-plus-components.kicad_sch       Sheet 2 — ATmega328P telemetry
    └── drone-flight-controller.kicad_pcb          Routed four-layer board
```

Project conventions and agent-facing context live in [CLAUDE.md](CLAUDE.md) and
[agents/](agents/).
