# 3-DOF Cycloidal Robotic Arm

A three-degree-of-freedom robotic arm driven by custom two-stage cycloidal gearboxes — designed from scratch using contracted cycloid profiles, 3D-printable housings, and absolute encoders on every joint. Built around spare EMAX RS2205 drone motors that needed a 500:1 reduction to be useful.

| | |
|---|---|
| **Motors** | EMAX RS2205 2300KV outrunner brushless × 3 |
| **Reduction** | 500:1 two-stage cycloidal (20:1 × 25:1) per joint |
| **Controllers** | MKS XDrive Mini (ODrive v3.6 clone) × 3 |
| **Computer** | Raspberry Pi 4 over CAN bus |
| **Reach** | ~830 mm (2 × 415 mm segments) |
| **Torque** | 10–15 Nm continuous, 30–44 Nm peak per joint |
| **Feedback** | AS5048A 14-bit magnetic (shoulder/elbow), TCRT5000 optical (base) |
| **Print material** | PLA+ or ABS |

![Arm — top view](assets/robotic_arm_1.png) ![Arm — angled view](assets/robotic_arm_2.png)

---

## Table of Contents

- [Cycloidal Gearbox](#cycloidal-gearbox)
- [Gripper](#gripper)
- [Encoder Systems](#encoder-systems)
- [Electronics](#electronics)
- [Rotating Base](#rotating-base)
- [Load Analysis](#load-analysis)
- [Print Notes](#print-notes)
- [Assembly Sequence](#assembly-sequence)
- [Testing & Troubleshooting](#testing--troubleshooting)
- [Bill of Materials](#bill-of-materials)
- [Quick Start](#quick-start)
- [Repository Structure](#repository-structure)

---

## Cycloidal Gearbox

All three actuators use an identical two-stage cycloidal gearbox. Each stage has two cycloidal discs phased 180° apart on a shared eccentric crankshaft, cancelling vibration at high input speeds. The profiles use **contracted cycloids** to prevent undercutting — K1 = 0.618 (Stage 1) and K1 = 0.722 (Stage 2) — keeping the discs printable on a standard FDM printer with a 0.4 mm nozzle.

![Gearbox internals](assets/robotic_arm_4.png)

### Stage 1 — 20:1

| Parameter | Value |
|-----------|-------|
| Disc lobes / housing pins | 20 / 21 |
| Housing rod diameter | 8 mm hardened steel |
| Housing pin pitch radius | 34 mm |
| Disc outer diameter | 62 mm |
| Disc thickness | 6 mm |
| Output pins | 6 × 6 mm on 41 mm PCD |
| Eccentricity | 1.0 mm |
| Centre bearing | 6901ZZ (12×24×6) |
| Flange bearing | 6809ZZ (45×58×7) |

### Stage 2 — 25:1

| Parameter | Value |
|-----------|-------|
| Disc lobes / housing pins | 25 / 26 |
| Housing rod diameter | 8 mm hardened steel |
| Housing pin pitch radius | 36 mm |
| Disc outer diameter | 66 mm |
| Disc thickness | 6 mm |
| Output pins | 6 × 6 mm on 43 mm PCD |
| Eccentricity | 1.0 mm |
| Centre bearing | 6901ZZ (12×24×6) |
| Flange bearing | 6809ZZ (45×58×7) |

### Axial Stack Height (One Gearbox)

| Layer | Thickness |
|-------|-----------|
| Motor bell adapter | 8 mm |
| Bottom retaining plate | 3 mm |
| Bottom spacer (S1) | 3 mm |
| Cycloidal disc 1 (S1) | 6 mm |
| Sandwich spacer (S1) | 2 mm |
| Cycloidal disc 2 (S1) | 6 mm |
| Top retaining plate | 3 mm |
| Bottom spacer (S2) | 3 mm |
| Cycloidal disc 1 (S2) | 6 mm |
| Sandwich spacer (S2) | 2 mm |
| Cycloidal disc 2 (S2) | 6 mm |
| Top retaining plate | 3 mm |
| Output flange + bearing | 8 mm |
| **Total (excl. motor)** | **~59 mm** |

### Two-Disc Balance

The eccentric crankshaft has two journals, both offset 1.0 mm but 180° apart. Disc 1 orbits in one direction while Disc 2 orbits opposite, cancelling lateral forces that would otherwise shake the arm apart at 34,000 RPM.

---

## Gripper

Dual-servo adaptive jaw gripper mounted at the elbow-side output flange. The finger arms sweep into a wide V with ridged inner faces, so the jaws progressively conform around objects up to ~80 mm diameter rather than pinching at a single point.

> **Gripper model by Tazer** — [patreon.com/TazerEngineering](http://patreon.com/TazerEngineering/posts/smart-gripper-149409228)

![Gripper](assets/robotic_arm_3.png)

| Parameter | Value |
|-----------|-------|
| Actuation | 2 × DS3225 servo, antagonistic |
| Stall torque | ~25 kg·cm @ 6.8 V |
| Control | GPIO PWM, 50 Hz, 500–2500 µs |
| Jaw travel | ~0–90° per finger |
| Power | 5 V BEC (not Pi onboard 5 V) |

---

## Encoder Systems

**Base joint** — TCRT5000 reflective IR sensors reading a printed 40-pair quadrature disc (160 CPR) on the output flange face. The base centre is occupied by the rotating platform, so there's no accessible shaft end for a magnetic encoder. Incremental — requires a homing routine at startup.

**Shoulder & elbow** — AS5048A 14-bit absolute magnetic encoder (16,384 CPR, 0.0219°/step). Reads a 6 mm diametrically magnetised neodymium magnet on the output shaft tip over SPI. Knows position on power-up — no homing needed.

---

## Electronics

```
Raspberry Pi 4 → CANable 2.0 → CAN bus (500 kbit/s) → MKS #1 (ID 0, base)
                                                       → MKS #2 (ID 1, shoulder)
                                                       → MKS #3 (ID 2, elbow)
4S LiPo → 20A fuse → 3× MKS XDrive Mini (direct)
                  → 5V BEC → Pi + encoders + gripper servos
```

### Critical Setup

- **ODrive firmware v0.5.1 only** — v0.5.6+ breaks motor output on these boards
- **Desolder R57** on every MKS board (CAN transceiver fix — forces standby mode otherwise)
- Set ghost axis (axis 1) → **CAN node ID 63** on each board to prevent bus conflicts

### ODrive Configuration

```python
# Motor
axis0.motor.config.motor_type = MOTOR_TYPE_HIGH_CURRENT
axis0.motor.config.pole_pairs = 7
axis0.motor.config.current_lim = 20
axis0.motor.config.calibration_current = 5

# AS5048A (shoulder/elbow)
axis0.encoder.config.mode = ENCODER_MODE_SPI_ABS_AMS
axis0.encoder.config.cpr = 16384

# TCRT5000 (base)
axis0.encoder.config.mode = ENCODER_MODE_INCREMENTAL
axis0.encoder.config.cpr = 160
axis0.encoder.config.bandwidth = 100
```

---

## Rotating Base

The base gearbox uses the same two-stage cycloidal drive but drives a horizontal rotating platform instead of an output shaft. A 100 mm lazy susan bearing carries all axial load (arm weight + payload); the six gearbox output pins handle torque only — clean load path separation prevents bending fatigue.

---

## Load Analysis

Worst case: arm fully extended horizontally.

| Load | Mass | Torque at Shoulder |
|------|------|--------------------|
| Arm segment 1 | ~250 g | 0.51 Nm |
| Elbow gearbox + motor | ~230 g | 0.94 Nm |
| Arm segment 2 + encoder | ~260 g | 1.53 Nm |
| Gripper | ~100 g | 0.77 Nm |
| Payload (target) | ~500 g | 4.07 Nm |
| **Total** | **~1.34 kg** | **~7.82 Nm** |

With 10–15 Nm continuous capacity, the shoulder has ~28% headroom.

---

## Print Notes

| Part | Orientation |
|------|-------------|
| Housing rings | Bore axis vertical (upright) |
| Cycloidal discs | Flat on bed |
| Retaining plates | Flat |
| Arm segments | Lengthwise, long axis horizontal |

**Tolerances:** FDM holes print 0.2–0.4 mm undersized. Test before printing full housings. Disc centre bore: print at 23.95 mm for 6901ZZ press fit. Output pin holes: print at 5.85 mm for 6 mm rod press fit.

**Lubrication:** Lithium-complex grease (Super Lube) on all housing rods. No petroleum-based lubricants on PLA — causes stress cracking.

---

## Assembly Sequence

**Stage 1:** Press 21 housing rods into ring → install bottom plate + bearing → 3 mm spacer → press 6901ZZ into disc 1, slide onto crankshaft journal 1 → 2 mm sandwich spacer → disc 2 on journal 2 → top plate + bearing → insert 6 output pins.

**Stage 2:** Repeat with Stage 2 housing (26 rods at 36 mm pitch, 25-lobe discs, 43 mm PCD).

**Motor:** Attach RS2205 via bell adapter. Stator bolts to housing; only the bell rotates.

---

## Testing & Troubleshooting

### First Power-On

1. Power BEC only. Confirm 5 V stable.
2. USB to each MKS — run odrivetool. Set axis 1 → ID 63. Assign node IDs.
3. Calibrate motors disconnected from gearbox.
4. Bring up CAN: `ip link set can0 up type can bitrate 500000`
5. Test each joint at low speed. Swap motor phases if direction is wrong.

### Common Issues

| Symptom | Fix |
|---------|-----|
| No CAN communication | Desolder R57 on all boards |
| Motor oscillates | Lower pos_gain |
| Gearbox grinding | Grease all 8 mm housing rods |
| Encoder erratic (AS5048A) | Reduce air gap to 1–2 mm |
| Encoder erratic (TCRT5000) | Reduce gap to 3 mm, shield from light |
| Output shaft drifts | Check CPR matches encoder config |
| Disc cracks at pin holes | Switch to ABS, 50%+ infill |

---

## Bill of Materials

| Category | Items | Est. Cost |
|----------|-------|-----------|
| Motors & controllers | 3× RS2205, 3× MKS XDrive Mini, CANable 2.0, CAN cables | ~$163 |
| Computing & power | Pi 4, SD card, 4S LiPo 5000 mAh, charger, BEC, XT60, fuse | ~$140 |
| Encoders | 2× AS5048A, magnets, 2× TCRT5000, resistors | ~$22 |
| Gripper | 2× DS3225 servo, extension cables | ~$44 |
| Bearings | 12× 6901ZZ, 12× 6809ZZ, lazy susan | ~$53 |
| Hardware | 141× 8 mm rods, 18× 6 mm rods, M3/M2/M4 fasteners, grease | ~$43 |
| Wiring | 14 AWG + 22 AWG silicone, heat shrink, connectors | ~$15 |
| **Total** | | **~$356** |

Full CSV with links: [robot_arm_parts_list.csv](robot_arm_parts_list.csv)

---

## Quick Start

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-venv can-utils -y
pip install odrive==0.5.1 python-can
sudo ip link set can0 up type can bitrate 500000
sudo ifconfig can0 up
```

```python
import odrive
odrv_base = odrive.find_any(path='can:can0:0')
odrv_shoulder = odrive.find_any(path='can:can0:1')
odrv_elbow = odrive.find_any(path='can:can0:2')
odrv_base.axis0.requested_state = AXIS_STATE_CLOSED_LOOP_CONTROL
```

---

## Repository Structure

```
├── assets/            # Images (4) and demo video
├── CAD/               # Full STEP assembly
├── PCB/               # Circuit board designs (TBD)
├── firmware/          # Firmware (TBD)
├── JOURNAL.md         # Build log
└── README.md
```

---

## License

Open engineering reference — use and modify freely.
