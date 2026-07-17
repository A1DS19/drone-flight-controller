# Drone Flight Controller

KiCad 10 project for a custom STM32F103R8T6-based drone flight controller with an ATmega328P
telemetry co-processor.

## Current state

Schematic capture is essentially complete across a two-page hierarchical schematic. The PCB layout
has not been started.

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

### Open items

The design is not ready for PCB layout or fabrication yet:

- The custom microSD symbol (U8) declares all twelve pins as `input`; the electrical types
  (power, bidirectional, passive) need correcting before ERC results are meaningful.
- `L1` in the VDDA filter still carries the invalid value `27nF`.
- No `PWR_FLAG` symbols exist, so ERC will warn on every rail.
- Footprints exist only for the major ICs; every passive, connector, crystal, switch, and LED
  still needs one. LCSC/MPN coverage is a single part (D3).
- The PCB file is empty: no outline, footprints, copper, or routing.
- Native ERC/DRC has never run — it requires a KiCad 10 `kicad-cli`.

See [agents/DESIGN_REVIEW.md](agents/DESIGN_REVIEW.md) for the full assessment and evidence basis.

## Open the project

Use KiCad 10 or newer and open:

`pcb/drone-flight-controller.kicad_pro`

The project was saved in KiCad 10 formats. KiCad 9 reports “Failed to load” for both the schematic
and board.

## Validate

With KiCad 10 installed (`kicad-cli` from KiCad 9 fails with “Failed to load”):

```sh
kicad-cli sch erc pcb/drone-flight-controller.kicad_sch \
  --output /tmp/drone-flight-controller-erc.rpt

kicad-cli pcb drc pcb/drone-flight-controller.kicad_pcb \
  --output /tmp/drone-flight-controller-drc.rpt
```

ERC and DRC are necessary but not sufficient. Verify symbol and footprint pin numbering against the manufacturer datasheet, synchronize the schematic and PCB, and review generated fabrication files before ordering.

## Repository layout

```text
.
├── CLAUDE.md                 Working brief, auto-loaded by Claude Code
├── AGENTS.md                 Codex repository instructions
├── README.md
├── .gitignore                KiCad locks, backups, and fabrication outputs
├── agents/                   Project docs (new-project skill layout)
│   ├── CONTEXT.md            Domain glossary
│   ├── roadmap.md            Goals and milestones
│   ├── decisions.md          Log of hard, costly-to-reverse decisions
│   └── DESIGN_REVIEW.md      Current engineering assessment
└── pcb/
    ├── datasheets/           Local manufacturer references (25 PDFs)
    ├── libs/                 Project libraries (custom microSD symbol, EasyEDA footprint + 3D model)
    ├── sym-lib-table         Project symbol-library table
    ├── fp-lib-table          Project footprint-library table (registers `easyeda`)
    ├── drone-flight-controller.kicad_pro
    ├── drone-flight-controller.kicad_sch          Sheet 1 — STM32 core + power
    ├── ATMEGA238P-plus-components.kicad_sch       Sheet 2 — ATmega328P telemetry
    └── drone-flight-controller.kicad_pcb          Layout (currently empty)
```

For agent-facing context, start with [CLAUDE.md](CLAUDE.md); it links into
[agents/](agents/) and the repository conventions.
