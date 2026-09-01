# hadirhany_alexeagles_electronics-
# Task 1 — Robot Power System Design

## Scenario

The robot's power system must supply two loads from a single 3S Li-ion battery:

- **Load A** — Songle SRD relay (9V coil)
- **Load B** — HC-SR04 ultrasonic sensor (5V)
- **Source** — 3S Li-ion battery, 12.6V full charge

---

##  1: Identify the Needs

| | Voltage needed | Current needed |
|---|---|---|
| Load A – Relay (Songle SRD-09VDC) | 9V nominal coil | 40mA (high-sensitivity, L-type) or 50mA (standard, D-type) |
| Load B – HC-SR04 | 5V | 15mA |
| Battery – 3S Li-ion | 12.6V full charge, sags to ~9.9V near empty (3.3V/cell cutoff) | — |

**Battery vs. need:** the battery (9.9V–12.6V across its discharge curve) is always *higher* than both the 9V and 5V requirements. This is a step-down problem for both rails, never step-up.

**Reverse-battery protection:** Required. Li-ion packs on simple 2-pin/JST connectors are easy to plug backwards, and a reversed 12.6V would instantly destroy a buck IC or LDO. A protection stage belongs right after the battery connector.

---

##  2: Choose the Components

- **Reverse protection → series Schottky diode** (e.g. 1N5819). Total system draw is only ~55–65mA, so the diode's ~0.3–0.4V drop and resulting heat are trivial. A MOSFET-based ideal-diode circuit would waste less voltage but isn't justified at these currents/cost targets.

- **12.6V → 9V (relay rail) → Buck converter**, not an LDO.
  The drop is large (up to 3.6V at full charge), and — more importantly — the battery can sag to 9.9V, leaving almost no headroom for a linear regulator to still hold a clean 9V. A buck converter (e.g. MP1584EN) stays efficient and regulated across the full battery discharge range.

- **9V → 5V (sensor rail) → LDO**, not a second buck.
  Current here is tiny (15mA) and the drop is only ~4V, so LDO dissipation is ~4V × 0.015A = 60mW — negligible heat. This avoids the cost, complexity, and EMI of a second switching regulator. AMS1117-5.0 or a 7805 both work.

---

##  3: Datasheet Check

- **MP1584EN (buck, 9V rail):** Input range covers 4.5–28V, comfortably spanning the 9.9–12.6V battery window. Rated output current is up to 3A — vastly more than the relay's ~50mA draw, confirming this part fits the load with large margin, not just "works."

- **AMS1117-5.0 (LDO, 5V rail):** Input up to 15V max, with typical dropout ~1.1V at 1A (much lower at 15mA). Feeding it from the regulated 9V rail leaves comfortable headroom to hold a clean 5V for the HC-SR04's 15mA draw.

---

##  4: LTspice Simulation

**Files in this repo:**
- `buck_sim.cir` — SPICE netlist for the buck stage
- `buck_schematic.svg` — labeled schematic diagram matching the netlist
- `Draft4.asc` — the actual LTspice schematic file
- `output_waveform.png` — screenshot of the settled simulation output

**Circuit:** An open-loop buck converter proving the topology:
- `V1` — 12.6V source (battery)
- `S1` — switch, driven by `V2` (a PULSE source: 0–5V, 1MHz, ~71% duty cycle, matching D = Vout/Vin = 9/12.6)
- `D1` — 1N5819 catch/freewheeling diode (cathode at the switch node, anode at ground — conducts only when the switch is open and the inductor pulls the switch node below ground)
- `L1` — 15µH inductor
- `Cin` — 10µF input capacitor
- `Cout` — 22µF output capacitor
- `Rload` — 180Ω, standing in for the relay's ~50mA draw at 9V

**Result:** After an initial LC startup transient (a spike followed by ringing — normal behavior for an open-loop switch with no soft-start), the output settles to a stable flat voltage by ~2ms. This confirms the buck topology is stable and delivers a regulated output to the relay rail.

**Note on accuracy:** This is an *open-loop* simulation with a fixed duty cycle, not a closed-loop model of the real MP1584 IC (which uses feedback to trim duty cycle automatically). The settled value is therefore an approximation of the real IC's regulated 9V output, not an exact match — a real MP1584 circuit would hold 9V precisely via its internal feedback loop.

---

## Summary

| Stage | Component | Why |
|---|---|---|
| Reverse protection | Schottky diode (1N5819) | Simple, low-loss at these currents |
| 12.6V → 9V | Buck converter (MP1584EN) | Large voltage drop + battery sag demands switching regulation |
| 9V → 5V | LDO (AMS1117-5.0) | Small drop, low current — linear is simpler and efficient enough |
