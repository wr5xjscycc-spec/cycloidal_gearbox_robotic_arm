# Build Journal — 3-DOF Cycloidal Robotic Arm

## Day 1 — Eccentric Crankshaft & Motor Enclosure
*2h 27m logged*

Started at the heart of the cycloidal drive: the eccentric crankshaft ("crack shaft"). This is the component that converts smooth motor rotation into the orbital motion that makes cycloidal drives work. Modeled the shaft with two offset journals 180° apart — each with a 1 mm eccentricity — so the two cycloidal discs per stage orbit in opposite directions and cancel each other's vibration. Also built the motor enclosure that holds the EMAX RS2205 securely and aligns it axially with the crankshaft input.

## Day 2 — Housing & Profile Fixes
*2h 26m logged*

Took a step back and cleaned up the design. Removed unnecessary geometry that was complicating the assembly and started on the housing ring for Stage 1. The housing ring holds 21 hardened steel rods at a 34 mm pitch radius — these are the fixed pins the cycloidal discs push against to generate the reduction. Getting the rod hole pattern right is critical: the housing bore, rod pitch radius, and rod diameter all interact to determine the cycloidal profile parameters.

*1h 26m logged*

Built the rollers (the steel housing pins) into the design. These are plain 8 mm hardened steel rods — no roller bearings, just grease-lubricated sliding contact. Simple and adequate at the output speeds we're running. Next up is the output pin flange — the part that catches the orbital motion and turns it into concentric rotation.

## Day 3 — Output Shaft & Hardware
*1h 35m logged*

Worked through the output pin flange geometry — six blind holes on a 41 mm PCD for Stage 1 that accept 6 mm output pin rods. These rods transfer torque from the orbiting cycloidal discs into the concentric output flange. The flange then drives either the next stage or the final arm joint.

*2h 9m logged*

Added all the M3 screw holes for the retaining plates that sandwich the disc stack together. The axial stack order is: bottom plate → 3 mm spacer → disc 1 → 2 mm sandwich spacer → disc 2 → top plate. Each plate bolts into the housing ring. Verified the full assembly in CAD — everything lines up, the discs orbit freely within the housing pins, and the output pins engage properly through both discs.

## Day 4 — Stage 1 to Stage 2 Coupling
*1h 3m logged*

Connected Stage 1 to Stage 2. The output flange of Stage 1 becomes the input crankshaft for Stage 2 — they share a common axis and bolt together rigidly. Designed the base flange that supports the output of Stage 2, which is where the actual arm segment attaches. This interface has to handle the full 500:1 accumulated torque.

## Day 5 — Two-Stage Gearbox Complete
*1h 11m logged*

The full two-stage gearbox is assembled in CAD. 20:1 first stage coupled to 25:1 second stage gives a clean 500:1 total reduction in a coaxial package about 59 mm tall (excluding the motor). The disc profiles use contracted cycloids — K1 = 0.618 for Stage 1 and 0.722 for Stage 2 — both safely under the undercutting limit. Next step is designing the actuator housing that wraps around this gearbox and mounts to the arm segments.

## Day 6 — Arm Structure Begins
*41m logged*

Started the arm segments. Going for hollow tapered profiles to keep weight down while maintaining stiffness — the arm has to support itself plus a payload at full horizontal extension, so every gram of segment weight directly loads the shoulder joint.

*1h 23m logged*

Integrated the encoder mounts into the arm design. The shoulder and elbow joints use AS5048A 14-bit absolute magnetic encoders reading a diametrically magnetized magnet on the output shaft tip — this gives true joint position after the full 500:1 reduction, not motor position. Set up the mounting brackets that hold the encoder boards at a consistent 1–2 mm air gap from the magnet face.

## Day 7 — Base Encoder Iteration
*1h 32m logged*

Working on the base encoder. The base joint is complicated because the output is a horizontal rotating platform — there's no accessible shaft end to mount a magnet on. Initial plan: tiny neodymium magnets embedded in the rotating platform read by a Hall sensor. Modeling the magnet pattern and sensor placement.

*1h 23m logged*

Set up the encoder mounts and wiring channels through the arm for the sensor cables.

*43m logged*

Scrapped the magnet idea — too fiddly and unreliable at the resolution needed. Switched to a TCRT5000 reflective IR sensor reading a printed black-and-white quadrature disc glued to the output flange face. 40 stripe pairs × 4 quadrature edges = 160 counts per revolution. Simpler, cheaper, and easier to iterate on since I can just reprint the disc pattern.

## Day 8 — Arm Connection to Base
*1h 27m logged*

Connected the arm assembly to the rotating base platform. The base uses a 100 mm lazy susan turntable bearing to carry all axial load (arm weight + payload), while the six gearbox output pins handle only rotational torque — clean load path separation. Designed the bolt pattern that transfers torque from the output flange into the platform.

## Day 9 — Base-to-Arm Joint
*53m logged*

Refined the interface between the base rotating platform and the first arm segment. The shoulder joint gearbox mounts vertically on the platform, with its output shaft horizontal — this transition from base rotation (vertical axis) to shoulder lift (horizontal axis) is the most mechanically complex interface in the whole arm.

## Day 10 — FOC Mount & Movement
*27m logged*

Added the mount for the MKS XDrive Mini (field-oriented controller) to the arm structure. Confirmed all joints have free range of motion without collision — the arm segments clear each other, the base rotates freely, and the encoder wiring doesn't snag.

## Day 11 — Final Assembly & Gripper
*2h 2m logged*

Everything assembled into one complete model. Integrated a free gripper model from online — dual-servo adaptive jaw design with ridged finger surfaces that conform around objects. The full arm: base → shoulder → elbow → gripper, three cycloidal gearboxes, three MKS controllers over CAN bus, all on a Raspberry Pi 4. Ready for prototype printing and commissioning.

---

**Total logged time:** ~20h 30m
