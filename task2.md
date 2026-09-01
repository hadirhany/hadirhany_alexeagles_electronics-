# Task 2 — PCB Component Identification


| Component type | Reference designator | Function & why it's needed |
|---|---|---|
| Capacitors | C27  | Decoupling/bulk filtering — smooths ripple on the 5V/10V/12V rails and supplies local charge for fast load transients so ICs don't glitch. |
| Resistors | R11  | Sets bias/feedback points for the regulator ICs (feedback divider) and provides pull-up/pull-down duties for signal lines. |
| Inductors | L1 | Energy-storage inductor for a switching (buck) regulator stage — steps the input voltage down to a lower rail efficiently, avoiding the heat loss of a linear regulator at this current. |
| Voltage Regulators | U4 (6-pin IC near the camera header / 5V rail) | Steps the board's input voltage down to a clean, regulated 5V rail for the camera header and other 5V logic. A second 3-pin linear regulator near C28 serves another local rail. |
| Connectors | J1 ("INPUT" barrel jack) | Main power entry point for the board. Other connectors (yellow terminal blocks, camera header) distribute power/signals to motors and the camera module. |
| Diodes | D1 (SMA package, cathode stripe, next to L1) | Positioned alongside the inductor — almost certainly the catch/freewheeling diode for the buck converter stage, or reverse-polarity protection on the input rail. |
