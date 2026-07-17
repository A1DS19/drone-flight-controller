# Roadmap

## Goal
A fabrication-ready two-layer STM32F103 flight controller: complete 3.3 V power tree, IMU and flight
sensors, ESC/motor and receiver/telemetry I/O, SWD debug, and a clean ERC/DRC + Gerber/DFM review.

## Milestones
- [x] Settle requirements: 12 V battery on XT-60, MPU-6050 IMU + TF-Luna LIDAR, RC receiver +
      nRF24L01P telemetry, three-pin ESC interface, USB + SWD debug, two-layer board. Board
      dimensions and mounting pattern get finalized during layout.
- [x] Add input protection and the power tree: XT-60 → SW4 master switch → `+12C` → LM7805 (`+5V`) →
      AMS1117-3.3 (`+3.3V`), with `Vdrive`/`VDDA` branches and a battery-sense divider. The current
      and noise budget is still undocumented.
- [x] Add BOOT0 default (R3 + boot-select jumper J2), NRST reset circuit (R1 pull-up, C9, SW1), and
      SWD debug header (J1).
- [x] Add the IMU and remaining flight-control interfaces: MPU-6050 on I²C, four ESC outputs,
      RC-receiver header, LIDAR, OLED, and SPI/I²C/USART expansion headers.
- [x] Add the telemetry subsystem: ATmega328P subsheet with nRF24L01P radio, CH340G USB-serial,
      microSD logging, and 5 V ↔ 3.3 V level shifting.
- [ ] Resolve the `L1` VDDA filter definition; add `PWR_FLAG`s to the rails; assign footprints + MPNs
      to every part (today: footprints on the ten major ICs only, one LCSC number).
- [ ] Run KiCad 10 ERC clean (needs a KiCad 10 `kicad-cli` — 9.0.9 cannot load the files), then
      update the PCB from the schematic.
- [ ] Define board outline, stackup, net classes, placement constraints, ground plane, and routing.
- [ ] Run KiCad 10 DRC, cross-domain/EMC review, BOM/MPN review, and Gerber/DFM checks before fabrication.

## Later / maybe
- Dedicated power, high-current, analog, clock, and controlled-impedance net classes.
- OSD or additional telemetry interfaces beyond the nRF24L01P link.
- Datasheet-level pin-map verification of the ATmega328P/nRF24/CH340/microSD subsheet (only the STM32
  power/reset/boot network has been formally cross-checked so far).
