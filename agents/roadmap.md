# Roadmap

## Goal
A fabrication-ready two-layer STM32F103 flight controller: complete 3.3 V power tree, IMU and flight
sensors, ESC/motor and receiver/telemetry I/O, SWD debug, and a clean ERC/DRC + Gerber/DFM review.

## Milestones
- [ ] Settle requirements: battery/input voltage, sensor set, receiver + telemetry protocols, ESC
      interface, USB/debug needs, board dimensions, mounting pattern, layer count.
- [ ] Add input protection and the 3.3 V power tree with a current and noise budget (no source exists today).
- [ ] Resolve the `L1` VDDA filter definition; add BOOT0 default (user-Flash pull-down), NRST/reset, and
      SWD debug circuitry; assign footprints + MPNs to every part.
- [ ] Add the IMU and remaining flight-control interfaces with explicit pin mapping and datasheet
      verification.
- [ ] Run KiCad 10 ERC clean, then update the PCB from the schematic.
- [ ] Define board outline, stackup, net classes, placement constraints, ground plane, and routing.
- [ ] Run KiCad 10 DRC, cross-domain/EMC review, BOM/MPN review, and Gerber/DFM checks before fabrication.

## Later / maybe
- Dedicated power, high-current, analog, clock, and controlled-impedance net classes.
- USB and telemetry/OSD interfaces beyond the minimum receiver link.
- NRST 0.1 uF reset-noise capacitor (add if the reset environment warrants; keep debugger able to pull NRST low).
