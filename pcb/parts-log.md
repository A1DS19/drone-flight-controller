# Parts selection log

Entry format per the pcb-source selection guide (§7).

### VDDA filter bead — L1: BLM15AG601SN1D (C76884)
- **Key specs:** 600 Ω @ 100 MHz ±25 %, 300 mA rated, 0.52/0.62 Ω DCR, 0402, −55~+125 °C
- **Price/stock:** $0.005 @ qty 1, stock 234,099, Extended
- **Rationale:** resolves the `27nF` placeholder; canonical ST-practice analog-rail bead; keeps
  the already-assigned `Inductor_SMD:L_0402_1005Metric` footprint. Paired change: `C8`
  100 nF → 10 nF so VDDA carries the datasheet-exact 10 nF + 1 µF (DS5319 Fig. 14 p. 36).
- **Alternatives considered:** Sunlord GZ2012D601TF (C1017, 0805, Basic, no feeder fee, matches
  FB1/FB2's size) — user preferred the smaller canonical Murata; deleting L1 for the
  datasheet-literal direct VDD tie — rejected, see decisions.md.
- **Design notes:** symbol stays `Device:L_Small` (a FerriteBead symbol swap risks the
  label-on-pin wiring for zero electrical gain); DCR drop at ~1.5 mA VDDA draw is < 1 mV.
- **Datasheet:** pcb/datasheets/C76884-BLM15AG601SN1D.pdf — p. 1 rating table row verified
  (600 Ω ±25 %, 300 mA, DCR 0.52/0.62 Ω); p. 3 temp range

### Master power switch — SW4: SS12D10G5 (C7431054)
- **Key specs:** SPDT vertical slide, non-shorting, THT 12.7 × 6.7 mm, AC 125 V / 2 A, −25~+70 °C,
  3,000 cycles, 2.2 mm travel, 250 gf
- **Price/stock:** $0.154 @ qty 1, stock 35,124, Extended (THT → hand-solder at JLC)
- **Rationale:** user-selected; retires the last named unsourced part. ~2 A rating vs the
  ~0.3–0.4 A logic-tree load through the LM7805 branch: ~5× margin. Latching (unlike the momentary
  tact parts considered earlier); THT for vibration robustness.
- **Alternatives considered:** G-Switch SS-12D11-G030 (3 A, right-angle, 10k cycles) and
  SS-12D01-G040 (1 A) — user preferred this one; 3k-cycle life is a non-issue for a master switch.
- **Design notes:** footprint `easyeda:SW-TH_SHOU-HAN_SS12D10G4` (EasyEDA shares the footprint
  name across the SS12D10 G-series lever variants — the part ordered is the **G5**). Pads 1/2/3 at
  4.8 mm pitch, **pad 2 = middle = common** per the datasheet schematic — matches the existing
  wiring (pin 1 `+12V` = ON throw, pin 2 `+12C` common, pin 3 unused throw, no-connect flagged).
  Model path set to `${KIPRJMOD}/libs/easyeda.3dshapes/`.
- **Datasheet:** pcb/datasheets/C7431054-SS12D10G5.pdf — p. 1 drawing (pins, 4.8/9.6 mm layout,
  non-shorting SPDT schematic), p. 3 §1.1 rating AC 125 V 2 A, §4.5 hand-solder 270 °C/4 s

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
