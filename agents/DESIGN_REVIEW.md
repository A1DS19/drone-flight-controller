# Drone Flight Controller Design Review

**Date:** 2026-07-11  
**Design format:** KiCad 10, single-sheet schematic, nominal two-layer PCB  
**Verdict:** Early capture only; not electrically complete and not ready for PCB layout or fabrication.

## Critical findings

| Severity | Finding | Evidence |
|---|---|---|
| Blocker | The PCB contains no outline, footprints, nets, tracks, vias, or zones. All 10 schematic components are absent from the PCB. | Raw PCB and schematic/PCB cross-analysis |
| Blocker | The design has no power input or 3.3 V regulator/source. `+3.3V` and `VDDA` are consumer-only rails. | Schematic topology analysis |
| Blocker | `BOOT0` is floating, so the MCU startup mode is not explicitly defined. Add an external pull-down and expose a deliberate override if the ROM bootloader is required. | Raw schematic; STM32 datasheet section 2.3.8 |
| Blocker | Nine of ten schematic components have no footprints, and no BOM line has a populated MPN. | Schematic BOM analysis |
| Warning | `L1` is an inductor symbol whose value is `27nF`, an invalid inductance unit, and it has no footprint or MPN. The VDDA filter therefore has unresolved electrical intent. | Raw schematic lines 2740–2772 |
| Warning | The design is not yet a flight controller: it lacks sensors, motor/ESC interfaces, receiver/telemetry I/O, SWD/programming access, clocks, protection, and a complete power tree. | Raw schematic component inventory |

## What is present

The schematic contains 10 physical components across 56 parsed nets:

| Type | Count | Notes |
|---|---:|---|
| MCU | 1 | STM32F103R8T6, LQFP-64 footprint assigned |
| Capacitors | 8 | Six 100 nF, one 4.7 uF, one 1 uF |
| Inductor | 1 | `L1`, currently valued `27nF`; intent unresolved |

Only three supply domains are named: `+3.3V`, `VDDA`, and `GND`. All GPIOs, `NRST`, and `BOOT0` are otherwise unconnected, with no explicit no-connect markers.

## MCU power network

```text
external 3.3 V source (missing)
          |
          +-- VDD x4 + VBAT
          |     +-- C1..C5: 100 nF bypass network
          |     `-- C6: 4.7 uF bulk
          |
          `-- L1: value/type unresolved --> VDDA
                                         +-- C7: 1 uF
                                         `-- C8: 100 nF

GND ----------------------------------------- VSS x4 + VSSA
```

The digital decoupling is directionally good. The included manufacturer datasheet, Figure 14 on page 36, calls for five 100 nF capacitors plus one 4.7 uF capacitor, with the 4.7 uF device connected to VDD3. The schematic has the corresponding quantities, but PCB placement cannot yet be checked.

For the analog rail, the same figure recommends VDDA decoupling of 10 nF plus 1 uF. The schematic uses 100 nF plus 1 uF after `L1`. This may work, but it differs from the cited network and should be an intentional choice supported by the selected filter component and expected ADC/PLL noise environment.

## Reset and boot

- The analyzer's “NRST missing pull-up” warning is a false positive. The STM32F103 has a permanent internal NRST pull-up of 30–50 kΩ (datasheet Table 39, page 66).
- The datasheet's recommended NRST protection adds 0.1 uF to ground (Figure 31, page 67). Add it if the expected reset-noise environment warrants it and ensure an external debugger can still pull NRST low.
- `BOOT0` needs an explicit default. Use a pull-down for normal boot from user Flash and provide a controlled way to pull it high only if system-memory boot is needed.

## PCB and EMC

The PCB analysis is not meaningful as a layout review because the board is empty. The EMC analyzer reports no ground plane and no stitching vias; these are true descriptions, but they are consequences of the absent layout rather than isolated routing defects. Its numerical risk score should not be interpreted as readiness.

Once placement begins, use a continuous ground plane, place each bypass capacitor adjacent to its associated VDD/VSS pair, keep the VDDA filter local to the MCU, and keep sensor/analog return paths away from noisy motor and switching-current loops.

## Project rules and compatibility

- The project defines only the default net class: 0.20 mm clearance, 0.20 mm track width, 0.60/0.30 mm via diameter/drill, and 0.50 mm copper-to-edge clearance.
- There are no dedicated power, high-current, analog, clock, or controlled-impedance net classes yet.
- The files were saved by KiCad 10.0 using schematic format `20260306` and PCB format `20260206`. Local KiCad CLI 9.0.9 could not load them, so native ERC and DRC were not run in this review.
- The root `AGENTS.md` makes the repository directly understandable to Codex and records safe edit/verification expectations. A project `.codex/config.toml` is intentionally omitted because no repository-specific model, sandbox, MCP, or hook setting is required.

## Analyses performed

- KiCad structural schematic analysis: performed
- KiCad full PCB/proximity analysis: performed
- Schematic/PCB cross-analysis: performed
- EMC pre-compliance heuristic: performed; limited by empty PCB
- Thermal heuristic: performed; zero power-dissipating parts could be assessed
- Raw schematic/PCB inspection: performed
- Included STM32 datasheet inspection: performed for power and reset/boot behavior

## Not performed / review limits

- Native ERC and DRC: not performed because the installed KiCad 9.0.9 cannot read KiCad 10 files.
- SPICE: not performed because no supported simulator is installed and there are no meaningful analog subcircuits yet.
- Gerber/DFM analysis: not applicable; no fabrication outputs or PCB geometry exist.
- Lifecycle/sourcing audit: not meaningful; MPN coverage is 0%.
- Full pin-level verification: limited to the MCU power pins and included datasheet. Interfaces and their parts have not been designed.
- Previous-review delta: no earlier review or analyzer run was present.

## Recommended next sequence

1. Settle requirements: battery/input voltage, sensor set, receiver and telemetry protocols, ESC interface, USB/debug needs, dimensions, mounting pattern, and layer count.
2. Add the input protection and 3.3 V power tree with a current and noise budget.
3. Fix the VDDA filter definition, add BOOT0/reset/debug circuitry, and assign footprints/MPNs.
4. Add IMU and remaining flight-control interfaces with explicit pin mapping and datasheet verification.
5. Run KiCad 10 ERC, then update the PCB from the schematic.
6. Define the outline, stackup, net classes, placement constraints, ground plane, and routing.
7. Run KiCad 10 DRC, cross-domain/EMC review, BOM review, and Gerber/DFM checks before fabrication.
