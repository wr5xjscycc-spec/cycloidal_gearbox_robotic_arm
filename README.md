# 3-DOF Cycloidal Robotic Arm

A three-degree-of-freedom robotic arm driven by custom two-stage cycloidal gearboxes — designed from scratch using contracted cycloid profiles, 3D-printable housings, and absolute encoders on every joint. Built around spare EMAX RS2205 drone motors that needed a 500:1 reduction to be useful.

| | |
|---|---|
| **Motors** | EMAX RS2205 2300KV outrunner brushless × 3 |
| **Reduction** | 500:1 two-stage cycloidal (20:1 × 25:1) per joint |
| **Controllers** | MKS XDrive Mini (ODrive v3.6 clone) × 3 |
| **Computer** | Raspberry Pi 4 over CAN bus (CANable 2.0) |
| **Reach** | ~830 mm (2 × 415 mm segments) |
| **Torque** | 10–15 Nm continuous, 30–44 Nm peak |
| **Feedback** | AS5048A 14-bit absolute magnetic (shoulder/elbow), TCRT5000 optical quadrature (base) |
| **Print material** | PLA+ or ABS |

![Arm assembly — top view](assets/robotic_arm_1.png)

*Full arm assembly showing the 3-DOF layout and segment geometry.*

![Arm assembly — angled view](assets/robotic_arm_2.png)

---

## Table of Contents

- [Cycloidal Gearbox Design](#cycloidal-gearbox-design)
- [Gripper Design](#gripper-design)
- [Encoder Systems](#encoder-systems)
- [Electronics Architecture](#electronics-architecture)
- [Rotating Base Design](#rotating-base-design)
- [Arm Structure & Load Analysis](#arm-structure--load-analysis)
- [Print & Fabrication Notes](#print--fabrication-notes)
- [Assembly Sequence](#assembly-sequence)
- [Testing & Commissioning](#testing--commissioning)
- [Troubleshooting](#troubleshooting)
- [Bill of Materials](#bill-of-materials)
- [Quick Start (Firmware)](#quick-start-firmware)
- [Critical Notes](#critical-notes)

---

## Cycloidal Gearbox Design

All three actuators use an identical two-stage cycloidal gearbox. Each stage consists of two cycloidal discs phased 180° apart on a shared eccentric crankshaft, eliminating the vibration that a single-disc design would produce at 34,000 RPM motor input.

![Cycloidal gearbox internals](assets/robotic_arm_4.png)

*Two-stage cycloidal gearbox section showing the dual-disc layout, eccentric crankshaft journals, and housing pin arrangement.*

### Profile Design

The disc profiles use a **contracted cycloid** rather than a full cycloid. A full cycloid would undercut for both stages because the ratio N × E / R exceeds 1.0. The shortening coefficients are:

- **Stage 1:** K1 = 0.618 (21 housing pins, 20 disc lobes)
- **Stage 2:** K1 = 0.722 (26 housing pins, 25 disc lobes)

Both are safely below the undercutting limit of 1.0. The contracted profile produces slightly shallower lobes (~2 mm deep vs ~2.3 mm for a full cycloid), reducing theoretical torque capacity by roughly 10–15% but eliminating undercutting entirely and keeping the profiles printable on a standard FDM printer with a 0.4 mm nozzle.

### Stage 1 — 20:1 Reduction

| Parameter | Value |
|-----------|-------|
| Lobe count (disc) | 20 |
| Housing pin count | 21 |
| Housing rod diameter | 8 mm, hardened steel |
| Housing pin pitch radius | 34 mm |
| Disc outer diameter | 62 mm |
| Disc thickness | 6 mm |
| Centre bore | 24 mm (6901ZZ bearing OD) |
| Output pin holes | 6 × 9 mm on 41 mm PCD |
| Output pin rods | 6 × 6 mm diameter × 25 mm |
| Eccentricity | 1.0 mm |
| Eccentric cam OD | 12 mm (6901ZZ bearing ID) |

### Stage 2 — 25:1 Reduction

| Parameter | Value |
|-----------|-------|
| Lobe count (disc) | 25 |
| Housing pin count | 26 |
| Housing rod diameter | 8 mm, hardened steel |
| Housing pin pitch radius | 36 mm |
| Disc outer diameter | 66 mm |
| Disc thickness | 6 mm |
| Centre bore | 24 mm (same bearing as Stage 1) |
| Output pin holes | 6 × 9 mm on 43 mm PCD |
| Output pin rods | 6 × 6 mm diameter × 25 mm |
| Eccentricity | 1.0 mm (same as Stage 1) |

### Two-Disc Balance Configuration

Each stage uses two identical cycloidal discs. The eccentric crankshaft has two journals, both with 1.0 mm offset from the shaft axis, pointing in exactly opposite directions — 180° apart. Disc 1 orbits in one direction while Disc 2 orbits in the opposite direction simultaneously, cancelling lateral inertial forces perfectly.

### Axial Stack Height (One Gearbox)

| Layer | Thickness |
|-------|-----------|
| Motor bell adapter | 8 mm |
| Bottom retaining plate | 3 mm |
| Bottom spacer (Stage 1) | 3 mm |
| Cycloidal disc 1 (Stage 1) | 6 mm |
| Sandwich spacer (Stage 1) | 2 mm |
| Cycloidal disc 2 (Stage 1) | 6 mm |
| Top retaining plate | 3 mm |
| Bottom spacer (Stage 2) | 3 mm |
| Cycloidal disc 1 (Stage 2) | 6 mm |
| Sandwich spacer (Stage 2) | 2 mm |
| Cycloidal disc 2 (Stage 2) | 6 mm |
| Top retaining plate | 3 mm |
| Output flange + bearing interface | 8 mm |
| **Total (excl. motor)** | **~59 mm** |

---

## Gripper Design

The end-effector is a **dual-servo adaptive jaw gripper** mounted at the elbow-side output flange. The finger arms sweep into a wide V with ridged inner faces, so the jaws progressively conform around a target rather than pinching at a single point.

> **Gripper model designed by Tazer** — used with permission. Check out his work at [patreon.com/TazerEngineering](http://patreon.com/TazerEngineering/posts/smart-gripper-149409228).

![Dual-servo adaptive gripper](assets/robotic_arm_3.png)

### Specifications

| Parameter | Value |
|-----------|-------|
| Actuation | 2 × servo, antagonistic/symmetric |
| Servo | DS3225 (or equiv. 25 kg·cm metal-gear digital) |
| Stall torque | ~25 kg·cm @ 6.8 V (~2.45 Nm) |
| Operating voltage | 4.8–6.8 V (5 V BEC supplied) |
| Control | GPIO PWM, 50 Hz, 500–2500 µs |
| Jaw travel | ~0–90° per finger |
| Max object diameter | ~80 mm |
| Finger contact | Ridged/serrated PLA+ or ABS |

### Wiring

| Servo Pin | Connects To |
|-----------|-------------|
| V+ (red) | 5 V BEC rail |
| GND (brown/black) | Common ground (Pi + BEC) |
| Signal (orange/yellow) | Pi GPIO (PWM-capable pin) |

> Tie the servo ground to the Pi ground to keep PWM signal levels referenced. Keep servo power off the Pi's onboard 5 V pins.

---

## Encoder Systems

The three joints use two different encoder technologies because of the mechanical constraints at each location.

### Base Joint — TCRT5000 Reflective Optical Encoder

The base uses two TCRT5000 reflective IR sensors reading a printed quadrature disc on the output flange face. The base centre is occupied by the rotating platform, so there's no accessible shaft end for a magnetic encoder.

| Parameter | Value |
|-----------|-------|
| Stripe pairs | 40 (80 total stripes) |
| Counts per rev (×4) | 160 |
| Angular resolution | 2.25° per count |
| Sensor gap | 3–5 mm from disc face |
| Pull-up resistors | 10 kΩ on each signal line to 5 V |
| Supply voltage | 5 V from BEC |

The base encoder is incremental — a homing routine must run at startup. The simplest approach is to rotate the base slowly toward a physical soft stop, detect the increased motor current, and set that position as zero.

### Shoulder & Elbow — AS5048A Magnetic Encoder

14-bit absolute rotary position sensor (16,384 positions per revolution, 0.0219° per step). Communicates over SPI to the MKS XDrive Mini J1 header.

| Parameter | Value |
|-----------|-------|
| Resolution | 14-bit (16,384 CPR) |
| Accuracy (linearised) | 0.05° |
| Interface | SPI |
| Supply | 5 V from BEC |
| Magnet | 6 mm diametrically magnetised neodymium |
| Air gap | 0.5–3 mm |

The magnet is glued into a recess at the centre of the output shaft tip. The encoder tolerates up to 0.5 mm lateral misalignment since it measures field direction, not field strength.

---

## Electronics Architecture

```
                         ┌─────────────────┐
                         │  Raspberry Pi 4  │
                         │  (trajectory, UI)│
                         └────────┬────────┘
                                  │ USB
                          ┌───────┴───────┐
                          │  CANable 2.0  │
                          └───────┬───────┘
                                  │ CAN bus (500 kbit/s)
            ┌─────────────────────┼─────────────────────┐
            │                     │                     │
    ┌───────┴───────┐    ┌───────┴───────┐    ┌───────┴───────┐
    │  MKS #1 (ID 0) │    │  MKS #2 (ID 1) │    │  MKS #3 (ID 2) │
    │  Base motor    │    │  Shoulder motor│    │  Elbow motor   │
    │  TCRT5000 enc  │    │  AS5048A enc   │    │  AS5048A enc   │
    └────────────────┘    └────────────────┘    └────────────────┘
            │                     │                     │
            └─────────────────────┼─────────────────────┘
                                  │
                         ┌───────┴───────┐
                         │  4S LiPo 14.8V│
                         │  → 20A fuse   │
                         └───────┬───────┘
                                 │
                         ┌───────┴───────┐
                         │  5V BEC       │
                         │  → Pi, encoders, servos
                         └───────────────┘
```

### CAN Bus

All three MKS XDrive Mini boards connect in a daisy-chain. 120 Ω termination is built into the CANable 2.0; install a second 120 Ω resistor across CANH/CANL on the third board.

**Critical:** The MKS XDrive Mini has a design flaw in the CAN transceiver circuit — resistor **R57** forces the chip into standby mode. **This resistor must be desoldered** on every board or CAN communication will not work.

### Power Distribution

| From | To | Wire | Fuse |
|------|-----|------|------|
| Battery XT60+ | 20A inline fuse | 14 AWG red | Yes |
| Fuse | 3× MKS XDrive Mini | 14 AWG red | Upstream |
| Battery | 5V BEC input | 14 AWG | None |
| 5V BEC | Pi GPIO pin 2 | 22 AWG | None |
| 5V BEC | All encoders VCC | 22 AWG | None |
| 5V BEC | Gripper servos VCC | 22 AWG | None |

### Node ID Configuration

| Board | CAN Node ID | Axis assignment |
|-------|------------|-----------------|
| MKS #1 (Base) | 0 | Axis 0 = base motor; Axis 1 → ID 63 |
| MKS #2 (Shoulder) | 1 | Axis 0 = shoulder motor; Axis 1 → ID 63 |
| MKS #3 (Elbow) | 2 | Axis 0 = elbow motor; Axis 1 → ID 63 |

The ghost axis (axis 1) on each board must be set to node ID 63 (listen-only) to prevent CAN bus conflicts.

### MKS Configuration (ODrive v0.5.1)

```python
# Motor (RS2205)
odrv0.axis0.motor.config.motor_type = MOTOR_TYPE_HIGH_CURRENT
odrv0.axis0.motor.config.pole_pairs = 7
odrv0.axis0.motor.config.current_lim = 20
odrv0.axis0.motor.config.calibration_current = 5
odrv0.axis0.motor.config.resistance_calib_max_voltage = 4

# Encoder — AS5048A (shoulder/elbow)
odrv0.axis0.encoder.config.mode = ENCODER_MODE_SPI_ABS_AMS
odrv0.axis0.encoder.config.cpr = 16384
odrv0.axis0.encoder.config.use_index = False

# Encoder — TCRT5000 (base)
odrv0.axis0.encoder.config.mode = ENCODER_MODE_INCREMENTAL
odrv0.axis0.encoder.config.cpr = 160
odrv0.axis0.encoder.config.use_index = False
odrv0.axis0.encoder.config.bandwidth = 100

# Control
odrv0.axis0.controller.config.control_mode = CONTROL_MODE_POSITION_CONTROL
odrv0.axis0.controller.config.pos_gain = 20
odrv0.axis0.controller.config.vel_limit = 10
odrv0.axis0.controller.config.vel_integrator_gain = 0.5
```

---

## Rotating Base Design

The base gearbox uses the same two-stage cycloidal drive as the arm joints, but with a horizontal rotating top platform instead of an output shaft.

### Load Path Separation

- **100 mm lazy susan turntable bearing** carries all axial load (arm weight + payload + platform)
- **Six gearbox output pin rods** carry only tangential (rotational) force
- No bending load on the output pins — prevents fatigue failure

The rotating platform is a flat disc ~120 mm diameter × 8 mm thick. Six blind holes on the underside match the Stage 2 output pin PCD (43 mm). The encoder disc is glued to the gearbox output flange face below the platform, with the TCRT5000 sensors mounted on a fixed bracket reading through the gap.

---

## Arm Structure & Load Analysis

Two identical hollow segments connect the three joints. Each measures 275 mm along its straight section with 70 mm radius rounded ends, giving a centre-to-centre distance of 415 mm per segment and ~830 mm total reach.

### Worst-Case Joint Loading (Arm Fully Extended Horizontally)

| Load Source | Mass | Distance from Shoulder | Torque |
|------------|------|----------------------|--------|
| Arm segment 1 | ~250 g | 207 mm | 0.51 Nm |
| Elbow gearbox + motor | ~230 g | 415 mm | 0.94 Nm |
| Arm segment 2 + encoder | ~260 g | ~600 mm | 1.53 Nm |
| Gripper + structure | ~100 g | ~780 mm | 0.77 Nm |
| Payload (target) | ~500 g | ~830 mm | 4.07 Nm |
| **Total shoulder demand** | | | **~7.82 Nm** |

With 10–15 Nm continuous torque capacity, the system has ~28% torque headroom at the shoulder before reaching thermal limits.

---

## Print & Fabrication Notes

### Material

- **PLA+** — acceptable for prototyping
- **ABS** — preferred for functional use (Tg ~100°C vs PLA's ~60°C) with better fatigue resistance
- **PETG** — intermediate option

### Print Orientation

| Part | Orientation |
|------|-------------|
| Housing rings | Bore axis vertical (upright) — layer lines perpendicular to roller loads |
| Cycloidal discs | Flat (disc face on bed) — layer lines parallel to disc face |
| Retaining plates | Flat |
| Arm segments | Lengthwise, long axis horizontal |

### Critical Dimensional Notes

- **8 mm housing rod holes:** FDM printers print holes 0.2–0.4 mm undersized. Print a test piece first.
- **24 mm disc centre bore:** Print at 23.95 mm for press fit on 6901ZZ bearing OD.
- **12 mm eccentric cam bore:** Print at 11.95 mm for H7 press fit on bearing.
- **6 mm output pin holes:** Print at 5.85 mm for tight press fit on 6 mm rod.

### Post-Processing

- Remove all support material carefully, especially inside housing bores
- Sand housing bore lightly with 400 grit if bearing doesn't seat smoothly
- Apply **lithium-complex grease** (Super Lube) to all 8 mm housing rods
- Do NOT use petroleum-based lubricants on PLA — causes stress cracking

---

## Assembly Sequence

Repeat for all three gearboxes. Each gearbox is identical.

### Stage 1 Sub-Assembly

1. Press 21 housing rods (8 mm × 25 mm) into Stage 1 housing ring at 34 mm pitch radius. Grease each rod.
2. Install bottom retaining plate. Bolt with M3 × 12 mm. Press one 6809ZZ bearing into plate centre.
3. Install 3 mm bottom spacer.
4. Press 6901ZZ bearing into disc 1 centre bore. Slide disc onto journal 1 of eccentric crankshaft.
5. Install 2 mm sandwich spacer.
6. Press 6901ZZ into disc 2. Slide onto journal 2 (180° opposite journal 1).
7. Install top retaining plate with second 6809ZZ bearing. Bolt to housing.
8. Insert 6 × 6 mm output pin rods through both discs into bottom/top flanges.

### Stage 2 Sub-Assembly

Repeat Stage 1 sequence with Stage 2 housing (26 rods at 36 mm pitch), Stage 2 discs (25 lobes, 43 mm PCD), and Stage 2 crankshaft.

### Motor Attachment

Attach RS2205 motor to Stage 1 input via bell adapter. Only the motor bell rotates — the stator is bolted to the housing base plate. Route three motor phase wires through housing wall channel to MKS XDrive Mini.

---

## Testing & Commissioning

### Pre-Power Checks

- [ ] R57 desoldered on all MKS boards
- [ ] All boards flashed with ODrive v0.5.1
- [ ] 20 A fuse installed on battery positive
- [ ] Common ground: battery negative ↔ all MKS boards ↔ BEC ↔ Pi ↔ encoders
- [ ] CAN bus terminated: 120 Ω at each end
- [ ] Encoder connections correct per joint type

### First Power-On

1. Power BEC only (not main battery). Confirm 5 V rail stable.
2. Connect via USB to each MKS board. Run odrivetool. Set ghost axis 1 → ID 63. Assign node IDs 0, 1, 2.
3. Run motor calibration (motors disconnected from gearbox). Verify resistance (~0.1 Ω) and inductance (5–15 µH).
4. Bring up CAN interface: `ip link set can0 up type can bitrate 500000`
5. Test each joint individually — command small position movement. If direction reversed, swap any two motor phase wires.
6. Run base homing routine. Verify reproducible home position.

### Thermal Monitoring

Monitor Stage 1 housing temperature after 30 s under moderate load. If uncomfortably hot (>50°C), reduce current limit.

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| CAN bus not communicating | R57 not removed | Desolder R57 on all boards |
| Motor oscillates/hunts | Position gain too high | Reduce `pos_gain` |
| Stage 1 housing hot | Current limit too high | Reduce `current_lim` to 15 A |
| Gearbox grinding noise | Insufficient lubrication | Disassemble, regrease 8 mm rods |
| Encoder erratic (AS5048A) | Air gap too large or magnet not diametric | Reduce gap to 1–2 mm |
| Encoder erratic (TCRT5000) | Ambient light or gap too large | Shield, reduce gap to 3 mm |
| Output shaft drifts | Wrong CPR setting | Verify CPR matches stripe count × 4 |
| Motor won't calibrate | Wrong firmware | Flash ODrive v0.5.1 only |
| Base won't home | Current threshold wrong | Reduce homing speed, increase threshold |
| Disc cracks at pin holes | PLA too brittle | Switch to ABS, increase infill to 50%+ |
| Arm can't hold position | Current limit too low | Increase up to 25 A incrementally |

---

## Bill of Materials

### Motors and Controllers

| Item | Qty | Source | Est. Cost |
|------|-----|--------|-----------|
| EMAX RS2205 2300KV brushless motor | 3 | AliExpress / Amazon | $15 each |
| MKS XDrive Mini (ODrive v3.6 clone) | 3 | AliExpress | $33–35 each |
| CANable 2.0 USB-CAN adapter | 1 | AliExpress | $10–12 |
| JST-GH 4-pin CAN cables (150 + 500 mm) | 3 sets | AliExpress | $5 pack |

### Computing and Power

| Item | Qty | Source | Est. Cost |
|------|-----|--------|-----------|
| Raspberry Pi 4 (4 GB) | 1 | Official / Amazon | $55–75 |
| 32 GB microSD card (Class 10) | 1 | Amazon | $8–10 |
| 4S LiPo battery 5000 mAh with XT60 | 1 | AliExpress (CNHL/GOLDBAT) | $22–28 |
| LiPo balance charger (ISDT Q6 Plus) | 1 | AliExpress | $28–35 |
| 5V 5A DC-DC step-down BEC | 1 | AliExpress (LM2596 / MP1584) | $3–5 |
| XT60 connectors (pairs) | 1 pack | AliExpress | $3–4 |
| 20A blade fuse + inline holder | 1 | AliExpress | $2–3 |

### Encoders

| Item | Qty | Use | Est. Cost |
|------|-----|-----|-----------|
| AS5048A 14-bit magnetic encoder module | 2 | Shoulder + elbow output shafts | $8–10 each |
| 6 mm diametrically magnetised neodymium magnet | 2 | One per AS5048A | $3 for 10 |
| TCRT5000 reflective IR sensor module | 2 | Base optical encoder (A + B channels) | $0.50 each |
| 10 kΩ resistors (pull-up) | 2 | TCRT5000 signal lines | Negligible |

### Gripper

| Item | Qty | Note | Est. Cost |
|------|-----|------|-----------|
| High-torque servo (25+ kg·cm stall) | 2 | DS3225 or equivalent | $15–25 each |
| Servo extension cables | 2 | Length to suit arm design | $3–5 |

### Bearings

| Bearing | Size | Qty | Purpose |
|---------|------|-----|---------|
| 6901ZZ deep groove ball bearing | 12 × 24 × 6 mm | 12 | Eccentric cam bearing (2 per stage × 2 stages × 3 gearboxes) |
| 6809ZZ deep groove ball bearing | 45 × 58 × 7 mm | 12 | Output flange support (same layout) |
| 100 mm lazy susan turntable bearing | 100 mm OD, round | 1 | Base rotating platform |

### Steel Rods and Fasteners

| Item | Spec | Qty | Purpose |
|------|------|-----|---------|
| 8 mm steel rod (housing pins) | Ø8 mm × 25 mm, hardened | 141 | Housing ring gear rods, all 3 gearboxes |
| 6 mm steel rod (output pins) | Ø6 mm × 25 mm | 18 | Output pin rods, all 3 gearboxes |
| M3 × 45 mm socket head cap screws | Fully threaded | 30+ | Assembly and retaining plates |
| M3 × 12 mm socket head cap screws | Standard | 40+ | Housing assembly |
| M2 × 4 mm grub screw | Set screw | 6 | Cam retention on motor shaft adapter |
| M4 × 10 mm socket head cap screw | Standard | 16+ | Lazy susan bearing mounting |

### Wiring

| Item | Qty | Purpose |
|------|-----|---------|
| 14 AWG silicone wire (red + black) | 2 m each | High-current power from battery to ODrives |
| 22 AWG multicolour silicone wire | 5 m | Signal wiring — encoders, CAN bus, servo |
| Heat shrink tubing assortment | 1 pack | All connections |
| JST-PH 2 mm connectors | 1 assortment | Encoder connections |

---

## Quick Start (Firmware)

```bash
# Raspberry Pi OS setup
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv can-utils -y
pip install odrive==0.5.1 python-can

# CAN interface (add to /etc/rc.local)
sudo ip link set can0 up type can bitrate 500000
sudo ifconfig can0 up

# Test CAN bus
candump can0
```

### Connecting to ODrives over CAN

```python
import odrive

odrv_base     = odrive.find_any(path='can:can0:0')
odrv_shoulder = odrive.find_any(path='can:can0:1')
odrv_elbow    = odrive.find_any(path='can:can0:2')

odrv_base.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

---

## Critical Notes

- MKS XDrive Mini requires **ODrive firmware v0.5.1 only** — v0.5.6+ breaks motor output
- **R57 must be desoldered** on every board for CAN to work
- Ghost axis (axis 1) must be set to **CAN node ID 63** to avoid bus conflicts
- Use **lithium-complex grease** — petroleum-based lubricants cause stress cracking in PLA
- Arm segments: print with **hollow interiors** to minimise weight
- Stage 1 housing: 21 rods at Ø8 mm × 25 mm, 34 mm pitch radius
- Stage 2 housing: 26 rods at Ø8 mm × 25 mm, 36 mm pitch radius

---

## License

This project is provided as an open engineering reference. Use and modify freely.
