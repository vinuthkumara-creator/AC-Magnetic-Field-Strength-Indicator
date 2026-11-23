# AC Magnetic Field Strength Indicator

[![Course](https://img.shields.io/badge/Module-EN2091%20Laboratory%20Practice%20%26%20Projects-blue)]()
[![University](https://img.shields.io/badge/University%20of%20Moratuwa-ENTC-maroon)]()
[![Band](https://img.shields.io/badge/Frequency%20Band-50%20Hz%20--%2015%20kHz-green)]()
[![Range](https://img.shields.io/badge/Field%20Range-5%20--%2090%20%C2%B5T-orange)]()

A handheld, battery-powered AC magnetic field strength detector designed to measure stray alternating magnetic fields from **5 µT to 90 µT** across three overlapping sensitivity ranges over a flat frequency response of **50 Hz to 15 kHz**.

Developed by **Team Analogico** — Department of Electronic & Telecommunication Engineering, University of Moratuwa, Sri Lanka.

---

## Table of Contents
- [Overview & Features](#overview--features)
- [System Architecture](#system-architecture)
  - [Design Requirements](#design-requirements)
  - [Signal Chain](#signal-chain)
  - [Gain & Signal-Level Budget](#gain--signal-level-budget)
  - [Frequency-Response Allocation](#frequency-response-allocation)
- [Component Selection (BOM)](#component-selection-bom)
- [PCB & Enclosure Design](#pcb--enclosure-design)
- [Simulation & Hardware Verification](#simulation--hardware-verification)
- [Repository Structure](#repository-structure)
- [Future Improvements & Applications](#future-improvements--applications)
- [Team & Acknowledgments](#team--acknowledgments)
- [References](#references)

---

## Overview & Features

Measuring low-level AC magnetic fields is essential for diagnosing transformer leakage, locating current-carrying conductors behind walls, and verifying electromagnetic compatibility (EMC) on PCBs. Because an inductive search coil produces an EMF proportional to both field strength ($B$) and frequency ($f$), this instrument uses an analog **lossy integrator** to equalize the frequency dependence—ensuring identical field amplitudes trigger the exact same LED indicator across the entire operating band.

* **Frequency-Independent Readout:** Flat response from **50 Hz to 15 kHz** (simulated $-3\text{ dB}$ passband: $18.6\text{ Hz} - 16.0\text{ kHz}$).
* **Three Selectable Ranges:**
  * Low: **5 – 25 µT**
  * Medium: **10 – 40 µT**
  * High: **20 – 90 µT**
* **9-LED Bar-Graph Display (Dot Mode):** Driven by an **LM3914** split into Green (1–3), Yellow (4–6), and Red (7–9) zones for low-power, clear visual feedback.
* **Isolated Handheld Probe:** A 1 mH radial inductor housed in a 3D-printed PETG pen probe connected via shielded SMA coaxial cable to prevent hand/body proximity interference.
* **Clean Dual-Rail Power:** Powered by two 9 V batteries ($\pm 9\text{ V}$) with zero switching converter noise.

---

## System Architecture

```
[AC Field B] ──► [Search Coil L1 (1 mH)] ──(e ∝ f·B)──► [Lossy Integrator U2 (Gain ∝ 1/f)]
                                                                   │ (∝ B)
                                                                   ▼
[Low-Pass Filter U9 (3rd Order, 241 kHz)] ◄── [RF Filter (339 kHz) & DC Block (0.72 Hz)]
      │
      ▼
[High-Pass Filter U1 (5th Order, 18.1 Hz)] ──► [Variable Gain U4] ──► [Precision Peak Detector U3/U5]
                                                                                  │ (DC)
                                                                                  ▼
                                                                      [LM3914 + 9 LEDs]
```

### Design Requirements

| Requirement | Target | Achieved By |
| :--- | :--- | :--- |
| **Frequency-independent reading** | Same field, same LED across band | Lossy integrator ($\text{Gain} \propto 1/f$) |
| **Operating bandwidth** | 50 Hz – 15 kHz, flat (measured) | 5th-order high-pass filter; coil $L_1$ with $R_{14}$ |
| **Field measurement range** | 5 – 90 µT in three ranges | Integrator, variable gain stage & display reference |
| **Out-of-band rejection** | Reject RF, DC drift & Earth-field motion | RF filter, 3rd-order LPF, and 5th-order HPF |
| **Adjustable sensitivity** | Weak and strong fields | Variable gain stage (panel potentiometer) |
| **Accurate low-level peak** | No silicon diode-drop error | Two-op-amp precision peak detector |
| **Clear visual indication** | 9 levels in 3 color bands | LM3914 configured in dot mode |
| **Portability & clean rails** | Battery powered, no switching noise | Two 9 V batteries ($\pm 9\text{ V}$ dual rail) |
| **Zero field distortion** | Non-ferromagnetic housing | 3D-printed PETG enclosure and probe |

### Signal Chain

| Stage | Main Parts | Function | Key Parameter |
| :--- | :--- | :--- | :--- |
| **Search Coil** | $L_1$ (1 mH radial inductor) | Magnetic flux to EMF | $\hat{e} = 2\pi f N A \hat{B}$ |
| **Lossy Integrator** | $U_2$, $R_{14}$, $C_{22}$, $R_{23}$ | Frequency compensation | $R_{14}C_{22} = 10\text{ µs}$, $f_c = 3.98\text{ Hz}$ |
| **RF Filter** | $R_1$ ($100\,\Omega$), $C_1$ ($4.7\text{ nF}$) | Attenuate RF pickup | $f_c = 339\text{ kHz}$ |
| **DC Block** | $C_2$ ($10\text{ µF}$ NP), $R_2$ ($22\text{ k}\Omega$) | Remove integrator DC offset | $f_c = 0.72\text{ Hz}$ |
| **Low-Pass Filter** | $U_9$, $R_3$–$R_7$, $C_3$–$C_5$ | 3rd-order Sallen-Key anti-RF | $f_{-3\text{dB}} = 241\text{ kHz}$, Gain $= 1.589$ |
| **High-Pass Filter** | $U_1$, $C_8$–$C_{12}$, $R_{10}, R_{11}, R_{15}, R_{16}, R_{22}$ | 5th-order motion/hum rejection | $f_{-3\text{dB}} = 18.1\text{ Hz}$, Gain $= 1.589$ |
| **Variable Gain** | $U_4$, $R_8$, $R_9$ (Potentiometer) | Range selection | $G = 1 + R_9/R_8$ ($\times 2$ reference) |
| **Peak Detector** | $U_3$, $U_5$, $D_1$, $D_2$, $C_7$, $R_{21}$ | Precision AC-to-DC peak hold | $\tau = R_{21}C_7 = 39\text{ ms}$ |
| **Display Driver** | LM3914, $R_{19}$, $R_{20}$, trimpots | Linear level-to-LED conversion | 9 active steps (Green / Yellow / Red) |

### Gain & Signal-Level Budget

For an identical magnetic flux density evaluated at **50 Hz** (2 mV induced EMF) and **1 kHz** (40 mV induced EMF) with the variable gain stage set to $\times 2$:

| Stage Output | Gain @ 50 Hz (V/V) | Gain @ 1 kHz (V/V) | Peak Level @ 50 Hz | Peak Level @ 1 kHz |
| :--- | :---: | :---: | :---: | :---: |
| **Coil EMF (Input)** | — | — | 2.00 mV | 40.0 mV |
| **Lossy Integrator** | 315 | 15.8 | 0.63 V | 0.63 V |
| **RF / DC Block + Low-Pass** | 1.58 | 1.58 | 1.00 V | 1.00 V |
| **High-Pass Filter** | 1.56 | 1.59 | 1.56 V | 1.59 V |
| **Variable Gain Stage** | 2.00 | 2.00 | 3.12 V | 3.18 V |
| **Peak Detector (DC Out)** | 1.00 | 1.00 | **3.12 V** | **3.18 V** |
| **Total (EMF to Peak Detector)** | **1559** | **79.5** | **Same LED Lit** | **Same LED Lit** |

### Frequency-Response Allocation

| Corner | Defined By | Corner Frequency | Role |
| :--- | :--- | :---: | :--- |
| **DC Block** | $C_2, R_2$ | 0.72 Hz | Blocks op-amp DC offset |
| **Integrator Lower Limit** | $R_{23}, C_{22}$ | 3.98 Hz | Sets DC gain ceiling ($-4000$) |
| **High-Pass ($-3\text{ dB}$)** | $C_8$–$C_{12}, U_1$ | **18.1 Hz** | **Lower passband edge** (blocks Earth-field motion) |
| **Coil Pole (Band Top)** | $L_1, R_{14}$ | **15.9 kHz** | **Upper passband edge** ($R_{14}/(2\pi L_1)$) |
| **Low-Pass ($-3\text{ dB}$)** | $U_9$ Sallen-Key | 241 kHz | Rejects SMPS and broadcast interference |
| **RF Filter** | $R_1, C_1$ | 339 kHz | Shunts high-frequency cable pickup |

---

## Component Selection (BOM)

All ICs are mounted in through-hole DIP sockets for easy servicing and replacement.

| Component | Part / Value | Key Specification | Engineering Justification |
| :--- | :--- | :--- | :--- |
| **Search Coil** | 1 mH Radial Inductor | Unshielded, ferrite drum core | High sensitivity in compact probe tip; forms 15.9 kHz pole with $R_{14}$ |
| **Op-Amps** | TL071 ($\times 6$) | JFET input, $13\text{ V/µs}$ slew rate | Negligible bias current with high-value resistors; ample bandwidth |
| **Display Driver** | LM3914 | 10-step linear comparator, regulated LED current | Uniform field steps; drives 9 LEDs without external ballast resistors |
| **Diodes** | 1N4148 | 4 ns reverse recovery, low leakage | Fast peak capture without discharging hold capacitor $C_7$ |
| **Calibration Trimmers** | $20\text{ k}\Omega$ (3296W) | 25-turn multiturn cermet | Fine calibration of $V_{\text{RLO}}$ and $V_{\text{RHI}}$ reference thresholds |
| **Probe Interconnect** | Coaxial Cable + SMA | Shielded, screw-locking | Prevents stray capacitive/inductive pickup along the cable |
| **Power Supply** | $2 \times 9\text{ V}$ Batteries | $\pm 9\text{ V}$ symmetrical rails | Zero switching noise or stray magnetic fields near sensor |

---

## PCB & Enclosure Design

* **Two-Layer PCB:** Compact mixed-technology (SMD + THT) layout integrating the preamplifier, multi-stage active filters, peak detector, and edge-mounted 9-LED bar-graph. Features top and bottom copper ground pours for EMI shielding, onboard test points, and multiturn calibration trimmers.
* **3D-Printed PETG Enclosure:** Split into a slender handheld **probe wand** (housing $L_1$) and a **main control unit** with isolated compartments for the PCB and dual 9 V batteries. PETG ensures high mechanical durability while remaining completely non-ferromagnetic and electrically insulating.

*(Place rendered PCB images, schematic screenshots, and enclosure photos inside `assets/images/` — see [Repository Structure](#repository-structure)).*

---

## Simulation & Hardware Verification

### Calibration Test Bench
Because a commercial gaussmeter was unavailable, a custom **solenoid verifier rig** was built using a transconductance amplifier driving a precision-wound air-core solenoid ($N = 200$ turns, diameter $D = 20\text{ mm}$, length $\ell = 50\text{ mm}$) in series with a $1\,\Omega$ current-sense resistor:

$$B = \frac{\mu_0 N I}{\sqrt{\ell^2 + D^2}} \approx 4.667\text{ mT/A} \implies B\,[\mu\text{T}] \approx 4.667 \times V_{\text{check}}\,[\text{mV}]$$

### Measured Thresholds (at 100 Hz)
Using a conservative conversion factor of $4.6\text{ µT/mA}$:

| Range Setting | Target LED | Measured Coil Current (mA) | Calculated Flux Density ($4.6 \times I$) |
| :--- | :---: | :---: | :---: |
| **5 – 25 µT** | LED 1 (Min) | 1.1 mA | **5.1 µT** |
| **5 – 25 µT** | LED 9 (Max) | 5.5 mA | **25.3 µT** |
| **10 – 40 µT** | LED 1 (Min) | 2.2 mA | **10.1 µT** |
| **10 – 40 µT** | LED 9 (Max) | 8.5 mA | **39.1 µT** |
| **20 – 90 µT** | LED 1 (Min) | 5.0 mA | **23.0 µT** |
| **20 – 90 µT** | LED 9 (Max) | 20.0 mA | **92.0 µT** |

---

## Repository Structure

```text
AC-Magnetic-Field-Strength-Indicator/
├── README.md                     # Project overview, architecture, BOM, and test results
├── .gitignore                    # Ignores LaTeX auxiliary build files and OS temp files
├── docs/
│   ├── report.tex                # Cleaned and structured LaTeX project report
│   └── references.bib            # Extracted BibTeX bibliography for clean compilation
├── assets/
│   └── images/                   # Place figures here (UOM.png, pcb.jpeg, enclosure*.jpeg, etc.)
├── hardware/
│   ├── pcb/                      # Place KiCad / Altium schematic, PCB layout & Gerbers here
│   └── enclosure/                # Place SolidWorks / Fusion 360 CAD & STL files here
└── simulation/                   # Place LTspice (.asc) simulation files here
```

---

## Future Improvements & Applications

### Hardware Roadmap
1. **Extended Dynamic Range (up to 500 µT):** Increase supply rails and rescale the non-inverting amplifier feedback network ($R_8, R_9$).
2. **Quantitative Digital Readout:** Interface an ADC and low-power LCD/OLED display alongside or in place of the LM3914 bar graph for direct numerical flux readouts in µT.

### Practical Applications
* **Navigation & Sensor Protection:** Detecting 5–20 µT stray fields that cause heading drift in drone/robot magnetometers or false triggering in Hall-effect switches.
* **PCB EMC Debugging:** Mapping stray flux around unshielded buck/boost inductors and validating mu-metal shielding placement.
* **Predictive Maintenance:** Non-invasive detection of current flow in traces/wires and early identification of transformer core saturation or motor winding faults.
* **Medical & Safety Screening:** Identifying magnetic exclusion zones around consumer electronics for pacemakers and checking high-frequency exposure against ICNIRP guidelines.

---

## Team & Acknowledgments

**Group Members (Department of Electronic & Telecommunication Engineering, University of Moratuwa):**

| Index No. | Name | Key Contributions |
| :---: | :--- | :--- |
| `240367U` | **K. A. D. V. Kumarasingha** | Team Lead, PCB Design, Schematic Design, Component Selection |
| `240559K` | **W. D. D. S. V. Rathnasiri** | PCB Design, Schematic Design, Component Selection |
| `240709X` | **M. P. Wickramarathna** | Enclosure Design, Schematic Design, Component Selection |
| `240723J` | **N. A. C. M. Wijerathna** | Enclosure Design, Schematic Design, Component Selection |

**Acknowledgments:**
We thank **Dr. Sampath Perera** (Module Coordinator, *EN2091 Laboratory Practice and Projects*), **Dr. Upeka Premaratne**, **Mr. Kavishka Abeywardana**, and the laboratory technical staff of the Department of Electronic & Telecommunication Engineering, University of Moratuwa, for their guidance and support.

---

## References

1. S. Tumanski, *"Induction coil sensors—a review,"* Measurement Science and Technology, vol. 18, pp. R31–R46, 2007.
2. P. Horowitz and W. Hill, *The Art of Electronics*, 3rd ed., Cambridge University Press, 2015.
3. J. Karki, *"Analysis of the Sallen-Key Architecture,"* Texas Instruments Application Report SLOA024B, 2002.
4. Texas Instruments, *"AN-31 Op Amp Circuit Collection,"* Application Report SNLA140B, 2013.
5. Texas Instruments, *"LM3914 Dot/Bar Display Driver,"* Datasheet SNVS761B, 2013.
6. Texas Instruments, *"TL07xx Low-Noise, FET-Input Operational Amplifiers,"* Datasheet SLOS080W, 2025.
7. D. J. Griffiths, *Introduction to Electrodynamics*, 4th ed., Pearson, 2013.
8. ICNIRP, *"Guidelines for limiting exposure to time-varying electric and magnetic fields (1 Hz to 100 kHz),"* Health Physics, vol. 99, no. 6, pp. 818–836, 2010.
