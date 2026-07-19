# Parts selection log

Entry format per the pcb-source selection guide (§7).

### Tact switches — SW1 (STM32 reset), SW2 (user button), SW3 (ATmega reset): TS24CA (C393942)
- **Key specs:** SPST momentary, 50 mA / 12 VDC, 1.6 N actuation, 20,000 cycles, side-actuated
  (right-angle), SMD 4.6 × 1.8 mm body with frame tabs
- **Price/stock:** $0.023 @ qty 1, stock 399,530, Extended (one-time feeder fee accepted)
- **Rationale:** user-selected; side actuation keeps the buttons reachable from the board edge
  once boxed, and one part number covers all three buttons
- **Alternatives considered:** Basic-library top-actuated tact switches (3×4 / 5.1×5.1 class) —
  rejected: would avoid the Extended feeder fee but are not edge-actuatable
- **Design notes:** footprint `easyeda:SW-SMD_2P-TS24CA` — pads 1/2 are the electrical terminals
  at 3.4 mm land spacing (3.35 ± 0.1 mm terminals per drawing), pads 3/4 are mechanical frame
  tabs left netless, two Ø0.7 mm locating holes at 1.7 mm pitch. **Not suitable for `SW4`** (the
  master power switch): momentary, and 50 mA is marginal for the logic tree — SW4 stays open.
- **Datasheet:** pcb/datasheets/C393942-TS24CA.pdf — 50 mA/12 VDC contact rating vs µA-scale
  reset/GPIO switching: fine; contact resistance ≤ 100 mΩ; single-pole single-loop matches the
  2-pin `Switch:SW_Push` symbol; bounce ≤ 10 ms (debounced by C9/C19/C24 networks already present)
