# CERN_EXTRACT_SIEMENS_KINEMATICS

A collection of Siemens SINUMERIK NC programs (MPF files) that read the
kinematic-chain configuration from machine data and system variables and write
it to a human-readable text report file.

Reference: *SINUMERIK 840D sl / SINUMERIK ONE — Machine Data and Parameters*
(`840Dsl_md_para_lists_man_1219_en-US`).

---

## Background — Kinematics Chain Definition in Siemens SINUMERIK

Siemens SINUMERIK (840D sl / SINUMERIK ONE) supports **multiple distinct
transformation types** to define the kinematic relationship between machine
axes and the programmed Cartesian tool path.  Each method stores its
geometric parameters in channel machine data (`$MC_` namespace, MD24xxx
range) and is activated by a dedicated G-code transformation command.

The transformation type is configured in `MD24100 $MC_TRAFO_TYPE_1` (and
MD24200/300/400 for slots 2–4).  Bits [3:0] encode an axis-sequence or
sub-variant; bits [11:4] identify the transformation group:

| # | Type | G-code | MD24100 range | Description |
|---|------|--------|---------------|-------------|
| 1 | **TRAORI** – head-head | `TRAORI` | 16 – 31 | 5-axis: both rotary axes on tool/spindle side |
| 2 | **TRAORI** – table-table | `TRAORI` | 32 – 47 | 5-axis: both rotary axes on workpiece/table side |
| 3 | **TRAORI** – mixed | `TRAORI` | 48 – 63 | 5-axis: one rotary on tool side, one on table side |
| 4 | **TRANSMIT** | `TRANSMIT` | 256 – 271 | Polar transformation (turning + milling, replaces Y-axis) |
| 5 | **TRACYL** | `TRACYL(r)` | 512 – 527 | Cylinder-surface transformation (unrolls cylinder to plane) |
| 6 | **TRAANG** | `TRAANG` / `TRAANG(α)` | 1024 – 1039 | Inclined-axis transformation (oblique slide to virtual Cartesian) |
| 7 | **Generic Kinematic Chain** | `TRAORI` | n/a (GKC) | SINUMERIK ONE: arbitrary serial chain of FIXED / TRANSLATION / ROTATION elements |

> **Axis-sequence examples for TRAORI** (bits [3:0]): 0=AB, 1=AC, 2=BA,
> 3=BC, 4=CA, 5=CB.  So `TRAFO_TYPE_1 = 20` means head-head with CA
> sequence (16 + 4 = 20).

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
| 24110 | `TRAFO_AXES_IN_1[0..n]` | Channel-axis input mapping (0 = unused) |
| 24120 | `TRAFO_GEOAX_ASSIGN_TAB_1[0..2]` | Geometry-axis → channel-axis assignment for active transformation |

### TRAORI – 5-axis orientation transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24500 | `TRAFO5_PART_OFFSET_1[0..2]` | Workpiece-carrier offset from reference point [mm] |
| 24510 | `TRAFO5_ROT_AX_OFFSET_1[0..2]` | Angular offset of orientation rotary axes at neutral position [deg] |
| 24520 | `TRAFO5_ROT_SIGN_IS_PLUS_1[0..1]` | Sign convention for each orientation axis (TRUE = not reversed) |
| 24530 | `TRAFO5_NON_POLE_LIMIT_1` | Limit angle for pole-range interpolation change [deg] |
| 24540 | `TRAFO5_POLE_LIMIT_1` | Max permitted end-angle deviation when switching to pole mode [deg] |
| 24550 | `TRAFO5_BASE_TOOL_1[0..2]` | Base tool offset at zero position [mm] |
| 24558 | `TRAFO5_JOINT_OFFSET_PART_1[0..2]` | Table-side joint offset (MIXED kinematics only) [mm] |
| 24560 | `TRAFO5_JOINT_OFFSET_1[0..2]` | Vector between the two rotary joints [mm] |

Orientation programming modes: `ORIWKS`, `ORIMKS`, `ORIAXES`, `ORIVECT`,
`ORIEULER`, `ORIRPY`

### TRANSMIT – polar transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24900 | `TRANSMIT_ROT_AX_OFFSET_1` | Angular offset of rotary axis at neutral position [deg] |
| 24905 | `TRANSMIT_ROT_AX_FRAME_1` | Whether offset is applied via transformation frame (0/1/2) |
| 24910 | `TRANSMIT_ROT_SIGN_IS_PLUS_1` | Sign of rotary axis (TRUE = not reversed) |
| 24911 | `TRANSMIT_POLE_SIDE_FIX_1` | Working area relative to pole (0=free, 1=positive X, 2=negative X) |
| 24920 | `TRANSMIT_BASE_TOOL_1[0..2]` | Base tool offset [mm] |

### TRACYL – cylinder surface transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24800 | `TRACYL_ROT_AX_OFFSET_1` | Angular offset of rotary axis at neutral position [deg] |
| 24805 | `TRACYL_ROT_AX_FRAME_1` | Whether offset is applied via transformation frame (0/1/2) |
| 24810 | `TRACYL_ROT_SIGN_IS_PLUS_1` | Sign of rotary axis (TRUE = standard direction) |
| 24820 | `TRACYL_BASE_TOOL_1[0..2]` | Base tool offset [mm] |
| — | `$P_TRACYL_CYLR` | Active cylinder radius at runtime [mm] (set in NC with `TRACYL(r)`) |

### TRAANG – inclined-axis transformation

| MD number | Variable (`$MC_`) | Description |
|-----------|-------------------|-------------|
| 24700 | `TRAANG_ANGLE_1` | Mechanical inclination angle α between inclined and reference axis [deg] |
| 24710 | `TRAANG_BASE_TOOL_1[0..2]` | Base tool offset [mm] |
| 24720 | `TRAANG_PARALLEL_VELO_RES_1` | Velocity reserve on parallel axis (0 = auto, >0 = fixed fraction 0..1) |
| — | `$P_TRAANG_ANG` | Active angle at runtime [deg] (overridable via `TRAANG(α)`) |

Kinematic equations (α = `TRAANG_ANGLE_1`):

```
Forward (virtual → physical):  Z_phys = Z_virt + Y_virt * sin(α)
                                Y_phys = Y_virt / cos(α)
Inverse (physical → virtual):  Z_virt = Z_phys - Y_phys * sin(α)
                                Y_virt = Y_phys * cos(α)
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
