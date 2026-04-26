# Single-Sided Linear Induction Motor (SLIM) — Propulsion System
### Avishkar Hyperloop | IIT Madras

A Single-Sided Linear Induction Motor (SLIM) designed as the primary propulsion system for a hyperloop pod. Unlike conventional rotary motors, a SLIM works by "unrolling" the motor flat — it pushes the pod forward by inducing currents in an aluminum track, with no physical contact between the motor and the track.

This repo documents the electromagnetic design, COMSOL FEM simulation results, and control architecture developed for Avishkar Hyperloop's propulsion subsystem.

---

## What is a SLIM?

A regular induction motor spins a rotor. A linear induction motor does the same thing but in a straight line — the "rotor" is replaced by a flat aluminum track, and the motor rides above it, generating both forward thrust and a vertical levitation/attraction force simultaneously.

This dual-force behavior makes SLIM particularly relevant for hyperloop applications where thrust and passive levitation can be achieved with a single electromagnetic system.

---

## Key Results

| Parameter | Value |
|---|---|
| Optimal drive frequency | **35 Hz** |
| Corresponding slip | **0.299** |
| Peak thrust force | ~3,500 N (x-component) |
| Peak normal force | ~4,400 N (y-component) |
| Pole pitch | 0.102 m |
| Operating current (baseline) | 80–100 A |
| Air gap | 6 mm |
| Aluminum track thickness | 6 mm |

> **Key finding:** Among 5 tested frequencies (25–65 Hz), 35 Hz produced the maximum thrust and normal force at an initial track velocity of 5 m/s. Higher frequencies reduced both thrust and levitation force despite higher slip values.

---

## Simulation Overview

All electromagnetic simulations were run in **COMSOL Multiphysics** using the AC/DC and Moving Mesh modules. The motor geometry was parameterized to allow rapid sweep studies.

### Design Parameters (COMSOL Model)

| Parameter | Value |
|---|---|
| Number of slots (n_s) | 28 |
| Slot width | 8.33 mm |
| Tooth width | 8.33 mm |
| Yoke height | 20 mm |
| Slot height | 47.73 mm |
| Drive frequency (f0) | 30 Hz (base) |
| Drive current (I0) | 100 A |
| Synchronous velocity | 6.12 m/s |
| Slip | 0.2 (base case) |
| Turns per coil (nc) | 72 |

---

## Parametric Studies Conducted

### 1. Frequency Sweep (25–65 Hz)
Simulated thrust and normal force at 5 frequency values with a fixed initial track velocity of 5 m/s and 100 A drive current. Result: 35 Hz is the optimal operating point for this pole pitch.

### 2. Slip Parametric Sweep
Swept slip from 0 to 0.5 at both 35 Hz and 45 Hz, with 80 A and 100 A excitation. Showed that thrust increases with slip up to a peak, then saturates — consistent with classical induction motor theory adapted for linear topology.

### 3. Current Magnitude Variation
Tested 80 A, 85 A, 90 A, and 95 A across frequencies from 35–65 Hz. Higher currents predictably increased both force components, with diminishing returns at high slip.

### 4. Zero Initial Velocity Case
Simulated the motor starting from standstill (slip = 1). At startup, peak thrust exceeded 5,000 N before settling to a steady-state of ~3,500 N — important for pod acceleration from rest.

---

## Design Principles (from Literature Review)

Derived from IEEE reference [5535183] and optimization study for low-speed SLIM (target: 70 km/h):

- **4-pole configuration** is the best trade-off — 2-pole suffers severe end effects, 6-pole becomes physically too large
- **Pole pitch range:** 250–350 mm for acceptable performance; below 200 mm, efficiency drops sharply
- **Slot-to-tooth ratio:** 3:2 (wide slots) reduces leakage flux and improves current loading
- **Aluminum track thickness:** minimum 5 mm; below this, energy consumption rises steeply
- **Fm/Fn ratio** (max-to-rated thrust): optimal range 1.5–2.5 for energy efficiency vs. weight trade-off
- **Winding type:** Double-layer, distributed, short-pitch winding with 3–6 slots per pole per phase

---

## Control Architecture — PMSM FOC

A Field-Oriented Control (FOC) strategy was designed in Simulink for the drive motor (PMSM) powering the SLIM inverter. The control flow:

1. Build DQ mathematical model of PMSM
2. Model PMSM in Simulink using DQ equivalent circuit
3. Design torque and speed calculation blocks
4. Implement Park and inverse Park transforms
5. Design dual PI controllers — one to hold Id = 0 (maximum torque per ampere), one to estimate required torque from reference speed
6. Build converter block for Iq from torque command
7. Assemble full FOC block
8. Design PWM comparator-based inverter model
9. Combine PMSM + FOC + Inverter for complete closed-loop simulation
10. Optimize PI gains until required torque-speed curves are achieved

See `control/pmsm_foc_flowchart.pdf` for the full implementation flowchart.

---

## Repository Structure

```
avishkar-slim-propulsion/
│
├── README.md
│
├── docs/
│   ├── SLIM_design_notes.md          # Key design principles from literature
│   └── SLIM_paper1_summary.md        # Optimization study notes (low-speed SLIM)
│
├── simulation/
│   ├── parameters.md                 # Full COMSOL parameter table
│   └── results/
│       ├── preliminary_results/      # Thrust & normal force vs frequency
│       ├── slip_parametric_sweep/    # Force vs slip (35Hz & 45Hz, 80A & 100A)
│       ├── current_variation/        # Force vs current (35–65 Hz)
│       └── initial_velocity_zero/    # Startup case: forces + B-field plot
│
└── control/
    └── pmsm_foc_flowchart.pdf        # Simulink FOC design flowchart
```

---

## Tools Used

- **COMSOL Multiphysics** — FEM electromagnetic simulation
- **MATLAB / Simulink** — FOC control system design
- **ANSYS Maxwell** — Cross-validation of magnetic field results

---

## Team

**Avishkar Hyperloop** — IIT Madras  
65-member multidisciplinary team competing at European Hyperloop Week.  
This work was developed under the Electromagnetics & Propulsion subsystem.

---

## References

1. IEEE Xplore — SLIM Design Values: https://ieeexplore.ieee.org/document/5535183
2. Low-speed SLIM optimization study (SUMT-based nonlinear programming, target: 70 km/h rail vehicle)
