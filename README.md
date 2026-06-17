# 3-DOF Cycloidal Robotic Arm

Complete engineering specification and build guide for a three-degree-of-freedom robotic arm driven by custom two-stage cycloidal gearboxes. The arm features two tapered, organically-profiled hollow segments and a dual-servo adaptive gripper with conforming jaws.

**Motors:** EMAX RS2205 2300KV outrunner brushless × 3
**Gearboxes:** 500:1 two-stage cycloidal (20:1 × 25:1)
**Controllers:** MKS XDrive Mini (ODrive v3.6 clone) × 3
**Computer:** Raspberry Pi 4
**Communication:** CAN bus via CANable 2.0
**Reach:** ~830 mm
**Continuous torque:** 10–15 Nm per joint

## Repository Structure

```
├── CAD/
│   ├── cycloidalarm.step             # Full CAD assembly (arm + gripper)
│   └── (cycloidal disc DXFs)*        # Cycloidal disc profiles (see Section 3.3)
├── PCB/                              # Circuit board designs (TBD)
└── firmware/                         # Motor controller firmware (TBD)
```

*DXF files for cycloidal disc profiles are to be generated from the parametric Python script.

## Key Specifications

| Parameter | Value |
|-----------|-------|
| Joints | 3 powered (base, shoulder, elbow) + gripper |
| Gear ratio per joint | 500:1 (20:1 × 25:1) |
| Motor | EMAX RS2205 2300KV × 3 |
| Max input speed | ~34,000 RPM (4S) |
| Max output speed | ~72 RPM per joint |
| Continuous torque | 10–15 Nm |
| Peak torque (brief) | 30–44 Nm |
| Reach | ~830 mm (2 × 415 mm segments) |
| Control interface | CAN bus (CANable 2.0 → Raspberry Pi 4) |
| Shoulder demand (worst case) | ~7.82 Nm (28% headroom) |
| Print material | PLA+ or ABS |

## Encoder Systems

- **Base joint:** TCRT5000 reflective optical encoder (quadrature ABZ, 160 CPR)
- **Shoulder & elbow:** AS5048A 14-bit absolute magnetic encoder (SPI, 16,384 CPR)

## Subsystems Overview

| Subsystem | Description | Status |
|-----------|-------------|--------|
| **Gearbox** | Two-stage contracted cycloidal (K1=0.618 / 0.722) | Designed |
| **Base** | 100 mm lazy susan bearing, optical encoder, load-path separation | Designed |
| **Arm structure** | Two hollow PLA/ABS segments with tapered organic profile, 415 mm centre-to-centre | Designed |
| **Gripper** | Dual-servo adaptive jaw gripper; swept-back fingers with ridged inner contact surfaces mold around objects; GPIO PWM | Designed |
| **Power** | 4S LiPo 5000 mAh → 5V BEC → Pi + servos, direct → ODrives | Designed |
| **Firmware** | ODrive v0.5.1, ODrive Python lib, SocketCAN | To be written |
| **PCB** | Motor controller breakout / sensor wiring | To be designed |

## Bill of Materials

### Motors and Controllers

| Item | Qty | Source | Est. Cost |
|------|-----|--------|-----------|
| EMAX RS2205 2300KV brushless motor | 3 | AliExpress / Amazon | $15 each |
| MKS XDrive Mini (ODrive v3.6 clone) | 3 | AliExpress | $33–35 each |
| CANable 2.0 USB-CAN adapter | 1 | AliExpress | $10–12 |
| JST-GH 4-pin CAN cables (150 + 500 mm) | 3 | AliExpress | $5 pack |

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
| 6901ZZ deep groove ball bearing | 12 × 24 × 6 mm | 12 (4 per gearbox) | Eccentric cam bearing (2 per stage × 2 stages) |
| 6809ZZ deep groove ball bearing | 45 × 58 × 7 mm | 12 (4 per gearbox) | Output flange support bearing (2 per stage × 2 stages) |
| 100 mm lazy susan turntable bearing | 100 mm OD, round | 1 | Base rotating platform support |

### Steel Rods and Fasteners

| Item | Spec | Qty | Purpose |
|------|------|-----|---------|
| 8 mm steel rod (housing pins) | Ø8 mm × 25 mm, hardened | 141 total | Housing ring gear rods, all 3 gearboxes |
| 6 mm steel rod (output pins) | Ø6 mm × 25 mm | 18 total | Output pin rods, all 3 gearboxes |
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

## Gripper Design

The end-effector is a **dual-servo adaptive jaw gripper** mounted at the elbow-side output flange. Unlike a simple parallel pincer, the finger arms are swept into a wide V and carry ridged inner faces, so the jaws progressively conform around a target rather than contacting it at a single point.

### Gripper Specifications

| Parameter | Value |
|-----------|-------|
| Actuation | 2 × servo, antagonistic/symmetric drive |
| Servo | DS3225 (or equiv. 25 kg·cm metal-gear digital) |
| Servo stall torque | ~25 kg·cm @ 6.8 V (~2.45 Nm) |
| Operating voltage | 4.8–6.8 V (5 V BEC supplied) |
| Control signal | GPIO PWM, 50 Hz, 500–2500 µs pulse |
| Jaw travel | ~0–90° per finger (open ↔ closed) |
| Max object diameter | ~80 mm (self-centering on cylindrical parts) |
| Finger contact face | Ridged / serrated PLA+ (or ABS) |
| Mounting | Bolts to elbow output flange |

### Design Features

- **Conformable finger geometry** — each jaw arm sweeps backward at an angle so that as the fingers close, they progressively wrap around the target object rather than pinching at a single contact point. The swept V-profile lets the gripper self-center on objects up to ~80 mm diameter before the ridged faces engage.
- **Ridged inner contact surfaces** — serrated gripping faces on the inner side of each finger increase friction and distribute clamping load across the object surface, improving grip on irregular, round, or smooth parts.
- **Dual servo actuation** — two high-torque servos (DS3225 or equivalent) drive the jaws symmetrically for a centered, balanced clamp. Each servo is commanded independently over GPIO PWM, allowing soft-close and force trimming in firmware.
- **Power & control** — servos run from the 5 V BEC rail (shared with the Pi), **not** from the 4S pack directly. PWM is generated by the Raspberry Pi 4 GPIO; budget ~1–2 A peak per servo under load when sizing the BEC.
- **Wrist behavior** — wrist rotation is not independently powered; gripper yaw/pitch is achieved through coordinated motion of the base, shoulder, and elbow joints.

### Wiring (Gripper)

| Servo Pin | Connects To |
|-----------|-------------|
| V+ (red) | 5 V BEC rail |
| GND (brown/black) | Common ground (Pi + BEC) |
| Signal (orange/yellow) | Pi GPIO (PWM-capable pin) |

> Tie the servo ground to the Pi ground to keep PWM signal levels referenced correctly. Keep servo power off the Pi's onboard 5 V pins — feed it from the BEC directly to avoid brownouts.

## Critical Notes

- MKS XDrive Mini requires **ODrive firmware v0.5.1 only** (v0.5.6+ incompatible)
- CAN transceiver resistor **R57 must be removed** on each board
- Ghost axis (axis 1) on each board must be set to **CAN node ID 63** to avoid bus conflicts
- Use **lithium-complex grease** on housing rods (not petroleum-based on PLA)
- Stage 1 housing rod count: 21 at Ø8 mm × 25 mm, pitch radius 34 mm
- Stage 2 housing rod count: 26 at Ø8 mm × 25 mm, pitch radius 36 mm

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

See the full specification in each subsystem's directory for detailed build instructions, BOM, and assembly sequence.

## License

This project is provided as an open engineering reference. Use and modify freely.
