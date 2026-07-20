# Parts selection log

Entry format per the pcb-source selection guide (§7).

### USB connectors — J3 (STM32 USB), J16 (CH340G USB): MicroXNJ (C404969)
- **Key specs:** micro-B 5P female, right-angle SMD, shell on 4 THT legs; 1.8 A/contact, 30 V DC,
  ≤50 mΩ contact, 5,000 mating cycles
- **Price/stock:** $0.037 @ qty 1, stock 144,146, Extended
- **Rationale:** the previous footprint had unknown provenance, no 3D model, and no MPN — this
  closes all three at once with a high-stock part
- **Alternatives considered:** keep the mystery footprint and hunt for a matching model — rejected
  (no provenance is no basis for fab); SHOU HAN MICRO 5.9ZB5.0 (C2765187) — different shell-leg
  arrangement from the incumbent pad pattern
- **Design notes:** footprint `easyeda:MICRO-USB-SMD_MICROXNJ_1`; shell legs renumbered 7/8/9 → "6"
  so all four land on the symbols' Shield pin (GND) — same convention as the earlier SH→6 fix.
  Signal pads 1–5 at 0.65 mm pitch match the drawing's Pin1…Pin5. Model path set to
  `${KIPRJMOD}/libs/easyeda.3dshapes/`. Reflow ≤250 °C/10 s (datasheet §5.6).
- **Datasheet:** pcb/datasheets/C404969-MicroXNJ.pdf — 1.8 A / 30 V vs USB 5 V loads: ample;
  pin map and land pattern verified against the p. 1 drawing

### Tact switches — SW1 (STM32 reset), SW2 (user button), SW3 (ATmega reset): TC-6610 -6*6*9-260g (C7528713) — supersedes TS24CA below
- **Key specs:** SPST momentary, 50 mA / 12 VDC, 2.6 N (260 gf), 50,000 cycles, 6×6 mm THT body,
  tall 9 mm round actuator, pin grid 6.5 × 4.5 mm (4-Ø1.5 holes suggested)
- **Price/stock:** $0.030 @ qty 1, stock 17,377, Extended + through-hole (feeder fee, hand-solder at JLC)
- **Rationale:** user-selected; the tall 9 mm top-actuated button stays reachable above surrounding
  parts / through the frame, and the 260 gf feel resists accidental presses
- **Alternatives considered:** TS24CA C393942 (previous pick) — replaced: side-actuated low-profile
  SMD didn't suit the tall-button requirement; SMD 6×6 tacts — rejected in favor of THT strength
  for a frequently-pressed panel-style button
- **Design notes:** **4-pin shorted-pair hazard**: datasheet circuit + PCB drawing show the pins
  6.5 mm apart are internally connected; switching is across the 4.5 mm span. The easyeda footprint
  numbers pads 1/2/3/4 distinctly, which with a 2-pin symbol would strap nets across a shorted pair
  (STM32 held in reset) — so the **official `Button_Switch_THT:SW_PUSH_6mm_H9.5mm`** is used
  instead: pads numbered 1,1,2,2 with same-numbered pads on the 6.5 mm (shorted) axis; verified
  drop-in for `Switch:SW_Push`. H9.5 is the closest official height variant to this 9.0 mm part
  (footprint identical; 3D model 0.5 mm taller). Official drill (1.1 mm class) is tighter than the
  datasheet's generous Ø1.5 suggestion but standard for the 0.7 mm flat pins.
- **Datasheet:** pcb/datasheets/C7528713-TC-6610_-6_6_9-260g.pdf — 12 V/50 mA vs µA reset/GPIO
  switching: fine; contact resistance ≤ 50 mΩ; insulation ≥ 100 MΩ @ 250 VAC; SPST single loop
  matches the 2-pin symbol; existing C9/C19/C24 debounce networks unchanged

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
