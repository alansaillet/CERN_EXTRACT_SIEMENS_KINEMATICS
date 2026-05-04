# SINUMERIK NC Language Functions Reference

This file documents **every SINUMERIK NC language function, built-in,
system variable, and language keyword** used in the MPF/SPF programs of this
repository, together with the exact Siemens manual and chapter from which
each item was taken.

> **Important note on documentation scope**
>
> The Siemens reference PDF already present in this folder
> (`840Dsl_md_para_lists_man_1219_en-US.pdf`) is a **machine-data parameter
> list** only.  It covers `$MC_` / `$MA_` / `$MD_` variables but does **not**
> document NC language syntax or system variables.  The NC language and
> system-variable sources are listed below with their respective document
> numbers.

---

## Siemens documentation used as sources

| Reference ID | Document title | Document number | Relevance |
|---|---|---|---|
| **[JP]** | SINUMERIK 840D sl / SINUMERIK ONE — Programming Manual: Job Planning | 6FC5398-2AP40-0BA3 (840D sl) · 6FC5398-5CP40-0BA3 (ONE) | All NC language constructs, file I/O, math functions, EXTCALL, GOSUB, DEF, PROC … |
| **[FU]** | SINUMERIK 840D sl / SINUMERIK ONE — Programming Manual: NC Fundamentals | 6FC5398-1AP40-0BA3 (840D sl) · 6FC5398-4CP40-0BA3 (ONE) | Axes, M-functions (M30), auxiliary functions, program structure |
| **[SV]** | SINUMERIK 840D sl / SINUMERIK ONE — System Variables | 6FC5398-8BP40-0BA3 (840D sl) · 6FC5398-8CP40-0BA3 (ONE) | `$P_` / `$AC_` / `$NK_` namespaces; `$PI` |
| **[MD]** | SINUMERIK 840D sl — Machine Data and Parameters (PDF in this repo) | 6FC5397-7AP40-6BA3 | `$MC_` / `$MA_` / `$MD_` machine data, MD24xxx range |

---

## File I/O functions

### `FILEOPEN`
| | |
|---|---|
| **Signature** | `DEF INT fd` `fd = FILEOPEN(path, mode)` |
| **Parameters** | `path STRING[120]` — full path on HMI filesystem; `mode INT` — 0 = write/create-truncate, 1 = append, 2 = read |
| **Return** | `INT` file descriptor ≥ 0 on success; negative value on error |
| **Minimum NCK** | NCK SW 4.5 (840D sl) |
| **Source** | **[JP]** Chapter "NC high-level language" → "File management" (sub-chapter "FILEOPEN") |
| **Used in** | `READ_ALL_KINEMATICS.MPF`, `01_TRAORI/READ_TRAORI_KINEMATICS.MPF`, `02_TRANSMIT/READ_TRANSMIT_KINEMATICS.MPF`, `03_TRACYL/READ_TRACYL_KINEMATICS.MPF`, `04_TRAANG/READ_TRAANG_KINEMATICS.MPF`, `05_GENERIC_KINCHAIN/READ_GENERIC_KINCHAIN.MPF`, `COMMON/_KIN_WRITE_UTILS.SPF` |

---

### `FILEWRITE`
| | |
|---|---|
| **Signature** | `FILEWRITE(fd, data)` |
| **Parameters** | `fd INT` — file descriptor from `FILEOPEN`; `data STRING` — text line to write (CR+LF appended automatically) |
| **Return** | none |
| **Minimum NCK** | NCK SW 4.5 (840D sl) |
| **Source** | **[JP]** Chapter "NC high-level language" → "File management" (sub-chapter "FILEWRITE") |
| **⚠ Verification note** | The file `840Dsl_md_para_lists_man_1219_en-US.pdf` in this repository does **not** contain NC language documentation and therefore cannot confirm or deny this function.  Verification must be done using document **[JP]** (6FC5398-2AP40-0BA3), chapter on "File management".  If your firmware version does not support `FILEWRITE`, see Siemens support note 109478084. |
| **Used in** | All MPF/SPF files — every `FILEWRITE(_fd, ...)` call |

---

### `FILECLOSE`
| | |
|---|---|
| **Signature** | `FILECLOSE(fd)` |
| **Parameters** | `fd INT` — file descriptor returned by `FILEOPEN` |
| **Return** | none |
| **Minimum NCK** | NCK SW 4.5 (840D sl) |
| **Source** | **[JP]** Chapter "NC high-level language" → "File management" (sub-chapter "FILECLOSE") |
| **Used in** | `READ_ALL_KINEMATICS.MPF`, all reader MPF files, `COMMON/_KIN_WRITE_UTILS.SPF` |

---

## String functions

### `TOSTRING`
| | |
|---|---|
| **Signature** | `TOSTRING(expr)` |
| **Description** | Converts a numeric (INT or REAL) or BOOL expression to its STRING representation.  For REAL values uses the default decimal notation; for INT a plain digit string. |
| **Return** | `STRING` |
| **Source** | **[JP]** Chapter "NC high-level language" → "String functions" → "TOSTRING" |
| **Used in** | All MPF files — every `TOSTRING(...)` call |

---

### `<<` (string concatenation operator)
| | |
|---|---|
| **Syntax** | `"prefix" << expr << "suffix"` |
| **Description** | Concatenates two STRING values.  The right-hand operand may be a STRING literal, a STRING variable, or the result of `TOSTRING(...)`.  The result is truncated to the declared maximum string length of the target variable. |
| **Source** | **[JP]** Chapter "NC high-level language" → "String operations" → "String concatenation operator <<" |
| **Used in** | All MPF/SPF files — every `_line = "..." << TOSTRING(...) << "..."` statement |

---

## Mathematical functions

### `COS`, `SIN`, `TAN`
| | |
|---|---|
| **Signatures** | `COS(angle_deg)` · `SIN(angle_deg)` · `TAN(angle_deg)` |
| **Argument unit** | **Degrees** (not radians) |
| **Return** | `REAL` |
| **Source** | **[JP]** Chapter "NC high-level language" → "Arithmetic functions" → "Trigonometric functions" |
| **Used in** | `04_TRAANG/READ_TRAANG_KINEMATICS.MPF` lines that compute `cos(α)`, `sin(α)`, `tan(α)` from `TRAANG_ANGLE_1` |

---

### `INT`
| | |
|---|---|
| **Signature** | `INT(real_value)` |
| **Description** | Truncates a REAL value to the integer part (rounds toward zero). |
| **Return** | `INT` |
| **Source** | **[JP]** Chapter "NC high-level language" → "Arithmetic functions" → "Type conversion INT" |
| **Used in** | `01_TRAORI/READ_TRAORI_KINEMATICS.MPF` — `_typeGroup = INT(_trafoType / 16) * 16` |

---

## Program structure and control

### `DEF`
| | |
|---|---|
| **Syntax** | `DEF <type> variable_name` or `DEF <type>[size] variable_name` |
| **Description** | Declares a local variable of type `INT`, `REAL`, `BOOL`, or `STRING[n]`. |
| **Source** | **[JP]** Chapter "NC high-level language" → "Variable definition" → "DEF" |
| **Used in** | All MPF/SPF files — all `DEF INT`, `DEF REAL`, `DEF BOOL`, `DEF STRING[n]` declarations |

---

### `PROC … SAVE`
| | |
|---|---|
| **Syntax** | `PROC name(param_list) SAVE` … `RET` |
| **Description** | Defines a named subprogram (sub-routine) with parameters.  `SAVE` causes the NC to save and restore the G-code group state on entry/exit. |
| **Source** | **[JP]** Chapter "NC high-level language" → "Subprograms" → "PROC" and "SAVE" |
| **Used in** | `COMMON/_KIN_WRITE_UTILS.SPF` — all `PROC KIN_xxx(...)` definitions |

---

### `RET` / `RET(value)`
| | |
|---|---|
| **Syntax** | `RET` or `RET(expr)` |
| **Description** | Returns from a `PROC` subprogram.  `RET(value)` passes a return value to the caller when the `PROC` is assigned to a variable (used by `KIN_OPEN_FILE`). |
| **Source** | **[JP]** Chapter "NC high-level language" → "Subprograms" → "RET" |
| **Used in** | `COMMON/_KIN_WRITE_UTILS.SPF` |

---

### `EXTCALL`
| | |
|---|---|
| **Syntax** | `EXTCALL "relative/path/program_name"` |
| **Description** | Calls an external MPF program by path relative to the active program's location on the NCK file system.  The file extension `.MPF` / `.SPF` is optional. |
| **Source** | **[JP]** Chapter "Subprograms" → "Subprogram call via EXTCALL" |
| **Used in** | `READ_ALL_KINEMATICS.MPF` — calls for each reader MPF |

---

### `GOSUB`
| | |
|---|---|
| **Syntax** | `GOSUB label_name` |
| **Description** | Calls an internal subroutine (defined by a label in the same program file).  Execution continues at `label_name:` and returns on `RET`. |
| **Source** | **[JP]** Chapter "Subprograms" → "Local subroutines with GOSUB" |
| **Used in** | `READ_ALL_KINEMATICS.MPF` — `GOSUB DECODE_TYPE` and `GOSUB SET_FLAGS` |

---

### `MSG`
| | |
|---|---|
| **Syntax** | `MSG("message text")` or `MSG("text" << variable)` |
| **Description** | Writes a text message to the operator panel (HMI) status/message line.  The message is visible during program execution. |
| **Source** | **[FU]** Chapter "Program execution" → "Operator messages" → "MSG" |
| **Used in** | All MPF files — status messages and error notices |

---

### `M30`
| | |
|---|---|
| **Syntax** | `M30` (on its own block) |
| **Description** | End of main program.  Stops program execution, resets the NC channel to its reset state, and optionally rewinds the program pointer. |
| **Source** | **[FU]** Chapter "Miscellaneous functions (M functions)" → "M30 — Program end" |
| **Used in** | All MPF files — last executable statement of every main program |

---

### `FOR … TO … ENDFOR`
| | |
|---|---|
| **Syntax** | `FOR counter = start TO end_value` … `ENDFOR` |
| **Description** | Count-controlled loop.  `counter` is incremented by 1 each iteration.  The body is not entered if `start > end_value`. |
| **Source** | **[JP]** Chapter "NC high-level language" → "Control structures" → "FOR" |
| **Used in** | `01_TRAORI`, `02_TRANSMIT`, `03_TRACYL`, `04_TRAANG`, `05_GENERIC_KINCHAIN` — axis-index and element-index loops |

---

### `IF … ELSE … ENDIF`
| | |
|---|---|
| **Syntax** | `IF condition` … `ELSE` … `ENDIF` |
| **Description** | Conditional execution.  `ELSE` branch is optional. |
| **Source** | **[JP]** Chapter "NC high-level language" → "Control structures" → "IF" |
| **Used in** | All MPF/SPF files |

---

## System variables

### `$P_PROG`
| | |
|---|---|
| **Type** | `STRING` (read-only) |
| **Description** | Full path and name of the currently executing NC program on the NCK file system. |
| **Source** | **[SV]** Section "$P_ — Program variables" → "$P_PROG" |
| **Used in** | All MPF files — written to report headers |

---

### `$AC_CHANNO`
| | |
|---|---|
| **Type** | `INT` (read-only) |
| **Description** | Number of the active NC channel (1-based).  Multi-channel machines may run this program in any channel. |
| **Source** | **[SV]** Section "$AC_ — Actual channel values" → "$AC_CHANNO" |
| **Used in** | All MPF files — written to report headers |

---

### `$P_TRAFO`
| | |
|---|---|
| **Type** | `STRING` (read-only) |
| **Description** | Name of the currently active transformation (e.g., `"TRAORI"`, `"TRANSMIT"`, `"TRAFOOF"` if none active). |
| **Source** | **[SV]** Section "$P_ — Program variables" → "$P_TRAFO" |
| **Used in** | All reader MPF files — written to the "Runtime status" section |

---

### `$P_TRAFRAME`
| | |
|---|---|
| **Type** | `FRAME` (read-only) |
| **Description** | Active transformation frame — the combined coordinate-system offset produced by the active transformation at runtime.  Read by TRANSMIT / TRACYL frame-flag interpretation. |
| **Source** | **[SV]** Section "$P_ — Program variables" → "$P_TRAFRAME" |
| **Used in** | `02_TRANSMIT`, `03_TRACYL` — referenced in comments explaining `ROT_AX_FRAME_1` |

---

### `$P_TRACYL_CYLR`
| | |
|---|---|
| **Type** | `REAL` (read-only) |
| **Description** | Active cylinder radius \[mm\] as set by the `TRACYL(r)` call in the part program. Zero when TRACYL is not active. |
| **Source** | **[SV]** Section "$P_ — Program variables" → "$P_TRACYL_CYLR" |
| **Used in** | `03_TRACYL/READ_TRACYL_KINEMATICS.MPF` |

---

### `$P_TRAANG_ANG`
| | |
|---|---|
| **Type** | `REAL` (read-only) |
| **Description** | Active TRAANG inclination angle \[deg\] as programmed by `TRAANG(α)` or taken from MD24700 when `TRAANG` (no argument) is used. Zero when TRAANG is not active. |
| **Source** | **[SV]** Section "$P_ — Program variables" → "$P_TRAANG_ANG" |
| **Used in** | `04_TRAANG/READ_TRAANG_KINEMATICS.MPF` |

---

### `$PI`
| | |
|---|---|
| **Type** | `REAL` (read-only constant) |
| **Description** | Mathematical constant π ≈ 3.141592653589793. |
| **Source** | **[SV]** Section "Predefined system variables" → "$PI" (also listed in **[JP]** Chapter "NC high-level language" → "Predefined constants") |
| **Used in** | `04_TRAANG/READ_TRAANG_KINEMATICS.MPF` — converts degrees to radians for trig functions |

> **Note:** SINUMERIK trig functions (`COS`, `SIN`, `TAN`) accept **degrees** as
> input.  The `$PI / 180.0` conversion in `READ_TRAANG_KINEMATICS.MPF` is
> therefore **only needed if you want to compute in radians yourself**.
> If calling `COS($MC_TRAANG_ANGLE_1)` directly you do *not* need the
> conversion — the degree argument is already correct.

---

## GKC system variables (`$NK_` namespace)

All `$NK_` variables are part of the **Generic Kinematic Chain** option,
available on SINUMERIK ONE ≥ V6.x and on 840D sl with the GKC option key.

| Variable | Type | Description | Source |
|---|---|---|---|
| `$NK_NUM_ELEMENTS` | `INT` | Total number of kinematic elements defined | **[SV]** "GKC system variables" → `$NK_NUM_ELEMENTS` |
| `$NK_NAME[n]` | `STRING[32]` | User-defined name of element `n` | **[SV]** "GKC system variables" → `$NK_NAME` |
| `$NK_T_KIN_TYPE[n]` | `INT` | Element type: 0=FIXED, 1=TRANSLATION, 2=ROTATION | **[SV]** "GKC system variables" → `$NK_T_KIN_TYPE` |
| `$NK_T_KIN_VEC[n, 0..2]` | `REAL` | Unit direction vector of the joint axis in parent frame | **[SV]** "GKC system variables" → `$NK_T_KIN_VEC` |
| `$NK_T_KIN_OFFSET[n, 0..2]` | `REAL` | Translation offset from parent origin to this element's origin \[mm\] | **[SV]** "GKC system variables" → `$NK_T_KIN_OFFSET` |
| `$NK_T_KIN_AXIS[n]` | `INT` | Associated machine-axis index (0 = none) | **[SV]** "GKC system variables" → `$NK_T_KIN_AXIS` |
| `$NK_T_KIN_CHAIN[n]` | `INT` | Chain ID: 0 = tool chain, 1 = workpiece chain | **[SV]** "GKC system variables" → `$NK_T_KIN_CHAIN` |
| `$NK_T_KIN_CTRL[n]` | `INT` | Control/status flags (reserved) | **[SV]** "GKC system variables" → `$NK_T_KIN_CTRL` |

---

## Channel machine data (`$MC_` namespace)

All `$MC_` variables used in the programs are documented in the **machine data
parameters list** (`840Dsl_md_para_lists_man_1219_en-US.pdf`, document
6FC5397-7AP40-6BA3) included in the `references/` folder.

Quick index of all `$MC_` variables used:

| MD number | `$MC_` variable | Used in | MD manual section |
|---|---|---|---|
| 24100 | `TRAFO_TYPE_1` | All readers | Chapter 2, section 24100 |
| 24200 | `TRAFO_TYPE_2` | `READ_ALL_KINEMATICS.MPF` | Chapter 2, section 24200 |
| 24300 | `TRAFO_TYPE_3` | `READ_ALL_KINEMATICS.MPF` | Chapter 2, section 24300 |
| 24400 | `TRAFO_TYPE_4` | `READ_ALL_KINEMATICS.MPF` | Chapter 2, section 24400 |
| 24110 | `TRAFO_AXES_IN_1[0..4]` | TRAORI, TRANSMIT, TRACYL, TRAANG readers | Chapter 2, section 24110 |
| 24120 | `TRAFO_GEOAX_ASSIGN_TAB_1[0..2]` | TRAORI, TRANSMIT, TRACYL, TRAANG readers | Chapter 2, section 24120 |
| 24500 | `TRAFO5_PART_OFFSET_1[0..2]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24500 |
| 24510 | `TRAFO5_ROT_AX_OFFSET_1[0..2]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24510 |
| 24520 | `TRAFO5_ROT_SIGN_IS_PLUS_1[0..1]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24520 |
| 24530 | `TRAFO5_NON_POLE_LIMIT_1` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24530 |
| 24540 | `TRAFO5_POLE_LIMIT_1` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24540 |
| 24550 | `TRAFO5_BASE_TOOL_1[0..2]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24550 |
| 24558 | `TRAFO5_JOINT_OFFSET_PART_1[0..2]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24558 |
| 24560 | `TRAFO5_JOINT_OFFSET_1[0..2]` | `READ_TRAORI_KINEMATICS.MPF` | Chapter 2, section 24560 |
| 24700 | `TRAANG_ANGLE_1` | `READ_TRAANG_KINEMATICS.MPF` | Chapter 2, section 24700 |
| 24710 | `TRAANG_BASE_TOOL_1[0..2]` | `READ_TRAANG_KINEMATICS.MPF` | Chapter 2, section 24710 |
| 24720 | `TRAANG_PARALLEL_VELO_RES_1` | `READ_TRAANG_KINEMATICS.MPF` | Chapter 2, section 24720 |
| 24800 | `TRACYL_ROT_AX_OFFSET_1` | `READ_TRACYL_KINEMATICS.MPF` | Chapter 2, section 24800 |
| 24805 | `TRACYL_ROT_AX_FRAME_1` | `READ_TRACYL_KINEMATICS.MPF` | Chapter 2, section 24805 |
| 24810 | `TRACYL_ROT_SIGN_IS_PLUS_1` | `READ_TRACYL_KINEMATICS.MPF` | Chapter 2, section 24810 |
| 24820 | `TRACYL_BASE_TOOL_1[0..2]` | `READ_TRACYL_KINEMATICS.MPF` | Chapter 2, section 24820 |
| 24900 | `TRANSMIT_ROT_AX_OFFSET_1` | `READ_TRANSMIT_KINEMATICS.MPF` | Chapter 2, section 24900 |
| 24905 | `TRANSMIT_ROT_AX_FRAME_1` | `READ_TRANSMIT_KINEMATICS.MPF` | Chapter 2, section 24905 |
| 24910 | `TRANSMIT_ROT_SIGN_IS_PLUS_1` | `READ_TRANSMIT_KINEMATICS.MPF` | Chapter 2, section 24910 |
| 24911 | `TRANSMIT_POLE_SIDE_FIX_1` | `READ_TRANSMIT_KINEMATICS.MPF` | Chapter 2, section 24911 |
| 24920 | `TRANSMIT_BASE_TOOL_1[0..2]` | `READ_TRANSMIT_KINEMATICS.MPF` | Chapter 2, section 24920 |

---

## How to obtain the referenced Siemens manuals

All document numbers above are Siemens order numbers that can be downloaded
free of charge from the Siemens Industry Online Support portal:

```
https://support.industry.siemens.com/cs/document/<document_number>
```

Examples:
- Job Planning (840D sl): https://support.industry.siemens.com/cs/document/109748031
- NC Fundamentals (840D sl): https://support.industry.siemens.com/cs/document/109748035
- System Variables (840D sl): https://support.industry.siemens.com/cs/document/109748034
