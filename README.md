# Drone Flight Controller

KiCad 10 project for a custom STM32F103R8T6-based drone flight controller.

## Current state

The repository is at the initial schematic-capture stage. It currently contains:

- one STM32F103R8T6 in LQFP-64;
- the MCU's digital and analog supply decoupling network;
- an empty PCB file with no outline, footprints, copper, or routing.

It does not yet contain the subsystems required for a functional flight controller: regulated power input, IMU, barometer or other sensors, ESC/motor outputs, receiver/telemetry interfaces, debug/programming connector, clocks, protection, or a completed PCB layout.

See [DESIGN_REVIEW.md](DESIGN_REVIEW.md) for the present blockers and evidence basis.

## Open the project

Use KiCad 10 or newer and open:

`pcb/drone-flight-controller.kicad_pro`

The project was saved in KiCad 10 formats. KiCad 9 reports “Failed to load” for both the schematic and board.

## Validate

With KiCad 10 installed:

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
├── DESIGN_REVIEW.md          Current engineering assessment
├── README.md
├── .gitignore                KiCad locks, backups, and fabrication outputs
├── agents/                   Project docs (new-project skill layout)
│   ├── CONTEXT.md            Domain glossary
│   ├── roadmap.md            Goals and milestones
│   └── decisions.md          Log of hard, costly-to-reverse decisions
└── pcb/
    ├── datasheets/           Local manufacturer references
    ├── drone-flight-controller.kicad_pro
    ├── drone-flight-controller.kicad_sch
    └── drone-flight-controller.kicad_pcb
```

For agent-facing context, start with [CLAUDE.md](CLAUDE.md); it links into
[agents/](agents/) and the repository conventions.
