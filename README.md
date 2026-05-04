# CERN_EXTRACT_SIEMENS_KINEMATICS

A collection of Siemens SINUMERIK NC programs (MPF files) that read the
kinematic-chain configuration from machine data and system variables and write
it to a human-readable text report file.

---

## Background — Kinematics Chain Definition in Siemens SINUMERIK

Siemens SINUMERIK (840D sl / SINUMERIK ONE) supports **five distinct
transformation types** to define the kinematic relationship between machine
axes and the programmed Cartesian tool path.  Each method stores its
geometric parameters in channel machine data (`$MC_` namespace, MD24xxx
range) and is activated by a dedicated G-code transformation command.

| # | Type | G-code | MD24100 value | Description |
|---|------|--------|---------------|-------------|
| 1 | **TRAORI** – head-head | `TRAORI` | 24 | 5-axis: both rotary axes on tool/spindle side |
| 2 | **TRAORI** – table-table | `TRAORI` | 40 | 5-axis: both rotary axes on workpiece/table side |
| 3 | **TRAORI** – mixed | `TRAORI` | 56 | 5-axis: one rotary on tool side, one on table side |
| 4 | **TRANSMIT** | `TRANSMIT` | 16 | Polar transformation (turning + milling, replaces Y-axis) |
| 5 | **TRACYL** | `TRACYL(r)` | 32 | Cylinder-surface transformation (unrolls cylinder to plane) |
| 6 | **TRAANG** | `TRAANG` / `TRAANG(α)` | 64 | Inclined-axis transformation (oblique slide to virtual Cartesian) |
| 7 | **Generic Kinematic Chain** | `TRAORI` | n/a (GKC) | SINUMERIK ONE: arbitrary serial chain of FIXED / TRANSLATION / ROTATION elements |

---

## Repository Structure

```
mpf/
├── READ_ALL_KINEMATICS.MPF           ← master dispatcher (run this first)
│
├── COMMON/
│   └── _KIN_WRITE_UTILS.SPF          ← shared file-write utilities (called by all readers)
│
├── 01_TRAORI/
│   └── READ_TRAORI_KINEMATICS.MPF    ← reads TRAORI 5-axis parameters
│
├── 02_TRANSMIT/
│   └── READ_TRANSMIT_KINEMATICS.MPF  ← reads TRANSMIT polar parameters
│
├── 03_TRACYL/
│   └── READ_TRACYL_KINEMATICS.MPF    ← reads TRACYL cylinder parameters
│
├── 04_TRAANG/
│   └── READ_TRAANG_KINEMATICS.MPF    ← reads TRAANG inclined-axis parameters
│
└── 05_GENERIC_KINCHAIN/
    └── READ_GENERIC_KINCHAIN.MPF     ← reads Generic Kinematic Chain ($NK_ variables)
```

---

## Machine Data Reference

### Common (all transformations)

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24100 | `TRAFO_TYPE_1` | Type of transformation 1 (see table above) |
| 24110 | `TRAFO_AXES_IN_1[0..4]` | Geometry-axis (virtual) input mapping |
| 24120 | `TRAFO_AXES_OUT_1[0..5]` | Machine-axis (physical) output mapping |

### TRAORI – 5-axis orientation transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24700 | `TRAFO5_PART_OFFSET_1[0..2]` | Workpiece pivot offset X/Y/Z [mm] |
| 24710 | `TRAFO5_ROT_AX_1_1[0..2]` | 1st rotary axis direction vector (unit) |
| 24720 | `TRAFO5_ROT_AX_2_1[0..2]` | 2nd rotary axis direction vector (unit) |
| 24730 | `TRAFO5_BASE_TOOL_1[0..2]` | Base tool direction at zero position (unit) |
| 24740 | `TRAFO5_ROT_AX_OFFSET_1[0..2]` | Known point on 1st rotary axis [mm] |
| 24750 | `TRAFO5_ROT_AX_OFFSET_1[3..5]` | Known point on 2nd rotary axis [mm] |
| 24760 | `TRAFO5_JOINT_OFFSET_1[0..2]` | Arm length between rotary axes [mm] |
| 24780 | `TRAFO5_NON_POLE_LIMIT_1` | Minimum tilt angle away from pole [deg] |
| 24782 | `TRAFO5_POLE_LIMIT_1` | Pole singularity handling limit [deg] |

Orientation programming modes: `ORIWKS`, `ORIMKS`, `ORIAXES`, `ORIVECT`,
`ORIEULER`, `ORIRPY`

### TRANSMIT – polar transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24520 | `TRANSMIT_ROT_AX_1` | Index of rotary (C) axis |
| 24530 | `TRANSMIT_ROT_SIG_1` | C-axis rotation sign (+1 / -1) |
| 24540 | `TRANSMIT_POLE_SIDE_FIX_1` | Pole crossing behaviour (0 = free, ±1 = locked) |
| 24545 | `TRANSMIT_STRAIGHT_LINE_TL_1` | Straight-line tool-centre compensation flag |

### TRACYL – cylinder surface transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24600 | `TRACYL_ROT_AX_1` | Index of rotary (C) axis |
| 24610 | `TRACYL_ROT_SIG_1` | C-axis sign (+1 / -1) |
| 24620 | `TRACYL_ROT_SIGN_IS_PLUS_1` | Sign convention flag |
| — | `$P_TRACYL_CYLR` | Active cylinder radius at runtime [mm] (set in NC with `TRACYL(r)`) |

### TRAANG – inclined-axis transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24820 | `TRAANG_ANGLE_1` | Mechanical inclination angle α [deg] |
| 24830 | `TRAANG_PARALLEL_AXIS_1` | Reference axis for angle measurement |
| 24840 | `TRAANG_AXES_IN_1[0..1]` | Virtual (programmed) axis mapping |
| 24850 | `TRAANG_AXES_OUT_1[0..1]` | Physical (machine) axis mapping |
| — | `$P_TRAANG_ANG` | Active angle at runtime [deg] (overridable via `TRAANG(α)`) |

Kinematic equations:

```
Forward:  Z_phys = Z_virt + Y_virt * sin(alpha)
          Y_phys = Y_virt / cos(alpha)
Inverse:  Z_virt = Z_phys - Y_phys * sin(alpha)
          Y_virt = Y_phys * cos(alpha)
```

### Generic Kinematic Chain (SINUMERIK ONE GKC option)

| Variable (`$NK_`) | Description |
|-------------------|-------------|
| `$NK_NUM_ELEMENTS` | Total number of kinematic elements |
| `$NK_NAME[n]` | User-defined element name |
| `$NK_T_KIN_TYPE[n]` | Element type: 0=FIXED, 1=TRANSLATION, 2=ROTATION |
| `$NK_T_KIN_VEC[n, 0..2]` | Joint axis unit vector in parent frame |
| `$NK_T_KIN_OFFSET[n, 0..2]` | Constant offset from parent origin [mm] |
| `$NK_T_KIN_AXIS[n]` | Associated machine-axis index (0 = none) |
| `$NK_T_KIN_CHAIN[n]` | Chain ID: 0 = tool chain, 1 = workpiece chain |

---

## How to Use

### 1. Load programs onto the NCK

Copy the entire `mpf/` tree to the NCK part-program directory, preserving
the sub-folder structure.  The sub-programs are called via `EXTCALL` relative
paths from `READ_ALL_KINEMATICS.MPF`.

### 2. Run the master program

In the HMI, select the program `READ_ALL_KINEMATICS` and run it in **MDA** or
**Auto** mode.  The program:

1. Scans all 4 transformation slots in the current channel.
2. Writes a summary to `/user/sinumerik/hmi/log/all_kinematics_summary.txt`.
3. Automatically calls the appropriate type-specific reader(s) based on the
   configured `TRAFO_TYPE`.
4. Optionally calls the GKC reader if `$NK_NUM_ELEMENTS > 0`.

### 3. Read the output files

All reports land in `/user/sinumerik/hmi/log/`:

| File | Content |
|------|---------|
| `all_kinematics_summary.txt` | Master list of all configured transformations |
| `traori_kinematics.txt` | TRAORI 5-axis: axis vectors, offsets, pivot points |
| `transmit_kinematics.txt` | TRANSMIT: rotary axis, sign, pole behaviour |
| `tracyl_kinematics.txt` | TRACYL: rotary axis, sign, active radius |
| `traang_kinematics.txt` | TRAANG: inclination angle, axis mapping, equations |
| `generic_kinchain.txt` | GKC: per-element name, type, vector, offset, axis |

---

## Platform Requirements

| Component | Minimum version |
|-----------|-----------------|
| SINUMERIK | 840D sl or SINUMERIK ONE |
| Firmware (NCK) | >= 4.5 (for `FILEOPEN` / `FILEWRITE` / `FILECLOSE`) |
| GKC reader | SINUMERIK ONE >= V6.x with GKC option |
| NC language | SINUMERIK NC High-Level Language (HLL) |

---

## License

CERN Open Hardware Licence v2 – Permissive
