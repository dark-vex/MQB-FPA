# Research tasks

Status as of 2026-05-28. Ordered by priority.

---

## DONE

- Extracted and partially decoded VauDaS DVF packages (`BL_LIBGatewV20`, `BL_LIBEPHVAMQB`)
- Identified all 164 VCDS CLB key groups and their constants; confirmed per-filename keystream
- Mapped complete FPA adaptation channel names from ODIS VCTool exports (Golf VIII GTI + R)
- Identified ZDC dataset part numbers: GTI = `V03935345NU`, Golf R = `V03935364D`
- Extracted FPA profile display names from ODIS GFS HSQLDB (`didb_GFS-v.od_OD.data`)
- Confirmed FPA parameterization is server-side ZDC; not stored in local ODIS package
- **Obtained Golf R FPA dataset bytes** → `research-inputs/golf8_r_5WA907530Q_V03935364D.bin` (4088 bytes, version "2033", MY2021)
- **Obtained Golf GTI FPA dataset bytes** → `research-inputs/golf8_gti_5WA907530J_V03935345NU.bin` (4088 bytes, version "2044", MY2021)
- Compared both datasets: only 13 non-header bytes differ; confirmed "common template, variant activation" model — `list_of_controls` is byte-for-byte identical between GTI and Golf R; new control IDs 0x4D/0x4E/0x4F/0x23 present in BOTH cars

---

## ~~1. Get a PHEV FPA dataset~~ — DONE

- Golf VIII GTI `V03935345NU` ✓ — 21 active controls, 5 profiles (Comfort/Sport/Offroad/Eco/Individual)
- Golf VIII R `V03935364D` ✓ — byte-for-byte identical list_of_controls; only 13 behavioural bytes differ
- Standard Golf VIII `V03935345ZH` ✓ — byte-for-byte identical to Golf R dataset
- **Golf VIII PHEV `V03935350BG`** ✓ — 15 active controls, 4 profiles; 538 bytes differ from Golf R

PHEV analysis complete. Key findings documented in `research-claude.md` section
"PHEV Dataset Parse — V03935350BG (2026-05-16)".

Remaining open questions for 0x4D/0x4E/0x4F (all corrected 2026-05-28):
- **0x4F**: No LC link (not_set). Always present. No J533 function bit written on mode change.
- **0x4E**: AGK(24) on combustion (GTI/R), SAK(14) on PHEV, ABSENT on US market GTI (2055) and 2056.
- **0x4D**: Appears twice per dataset. LC assignments differ by version — 2033/2044: MO+ToS_L; 2056: GE+DR; 2031: GE+MO.
- New versions 2055 (GTI-USA) and 2056 (Golf R RDM stock OEM) analyzed 2026-05-28.

**Also confirmed**: 2056 stock OEM has 18 controls, Race/0x10/0x11 profiles, RDM enabled (0x0803=0x43), all returns-to-Sport after restart.

---

## ~~2. Get a non-performance, non-PHEV Golf VIII reference pair~~ — CLOSED

**CLOSED 2026-05-27**: Golf 8 1.4 TSI (version 2033) is already available in `ClaudeDataset-investigation/`
and is byte-for-byte identical to the Golf R dataset. The standard-variant hypothesis was falsified —
`0x4D/0x4E/0x4F` are universal platform features, not optional-hardware-dependent. No further
standard-Golf collection needed for this purpose.

**Note**: The 14TSI is a Chinese-market vehicle (no telematics module, address 8107 absent).
FPA_Funktion_AFS/AGK/SAK = not active on this car (no steerable headlights, no active grille
shutters, no interior sound actuator). J533 SW/HW identical to GTI (`5WA907530J`/`5WA907530C`).

---

## ~~3. Export the ZDC parameter schema from ODIS-E~~ — CLOSED (wrong platform)

**CLOSED 2026-05-27**: `SET_FPA_MOD` is specific to the 3Q0 gateway (older MQB platform, Golf VII era).
It does not exist on `EV_GatewMQB2020` (Golf VIII / MQB Evo). This approach cannot be used
to label unknown byte offsets in the MQB2020 FPA dataset.

The remaining path to naming `unknown_*` fields is **Task 4** (before/after dataset pairs).

---

## ~~4. Produce controlled before/after dataset pairs~~ — SUBSTANTIALLY DONE

Six version-2056 dataset variants placed in `research-inputs/vctool/` provided the
before/after pairs needed to map the unknown button and profile regions.

Key results (see research-claude.md section "Version 2056 Dataset — MQB37WXRXGOLF"):

- **New control IDs 0x50 and 0x51** confirmed (appear twice each in list_of_controls)
- **0x0803** (`unknown_button_data_802[1]`) = Remember Drive Mode button enable flag
- **0x07E2/0x07E6/0x07EA** = `button_profile_lists[3/4/5]` first bytes (profile cycling lists)
- **ESC ring variants** zero out 0x51 from list_of_controls[6] and [9]
- **control_is_reset** bitmask expansion for 0x50, 0x51, Exhaustflap, Gearbox, Enginepower
- **Brakecontrol (ESC) profile settings** at list_of_grouped_controls[7] confirmed

**Still needed** for complete offset mapping:
- Before/after for `unknown_button_data_7F6` through `unknown_button_data_81A` blocks
- Before/after to map the PHEV section (0xAA3–0xFF3) — needs a PHEV with ODIS-E bench
- Before/after for AID banner, mode_light_on, and unknown_845_table
- The 2056 dataset is for MQB37WXRXGOLF (Golf R RDM variant); some offsets confirmed but
  not all unknown fields are resolved

---

## ~~5. Export gateway long coding with labels~~ — DONE

ODIS-E session export captured for Golf R J533 (5WA907530Q, EV_GatewMQB2020 v005011):
- 48-byte long coding (RDID 006) with all byte values and `[LO]_` field labels
- `$0C68` FPA_Funktion adaptation table (RDID 007) for both GTI and Golf R
- GTI-vs-R coding diff confirmed at bytes 15/16/20/22
- 3 wrong template placeholders (0x14/0x1F/0x20) corrected in `fpa_dataset.bt`

See `research-claude.md` section "Golf R Gateway ODIS-E Ground Truth (2026-05-26)"
and raw sources in `research-inputs/odis/`.

---

## ~~6. Check for plaintext label files on the ODIS-E Windows host~~ — CLOSED

Checked 2026-05-26: `C:\ProgramData\SAE\ODIS\data\Labels\` does not exist on this ODIS-E installation. Path is not present; no plaintext label cache available.

---

## ~~7. Update `fpa_dataset.bt` with newly confirmed control names~~ — DONE

All confirmed ODIS function names (`VAQ`, `ALR`, `ToS_L`, `ToS_Q`, `ESP`, `eBKV`, `AGK`, `AFS`, `AMB`, `mFDR`, `ESH`, `HDC`, `RWB`, `MO_BZS`) are present in the long-coding/FPA function decoder. ToS_L/ToS_Q labels corrected to "longitudinal dynamics" / "lateral dynamics" on `fix/fpa-long-coding-names` branch (2026-05-27).

Important distinction: `0x4D/0x4E/0x4F` are still unknown `list_of_controls` IDs, not the same namespace as `$0C68` FPA function IDs.

Profile names still need byte values confirmed from a real dataset before adding to template.

---

## ~~8. Collect one matched VCDS autoscan + dataset pair~~ — SUBSTANTIALLY DONE

**Completed 2026-05-28 via OBDEleven + RDID 006 string (GTI, ZDC V03935345NU).**

Obtained from user's Golf VIII GTI:
- Full $0C68 FPA_Funktion adaptation state (all 39 channels via OBDEleven screenshots)
- RDID 006 48-byte long coding string: `00 00 80 8D 0F 00 00 00 00 00 00 00 01 03 03 32 53 01 60 09 02 00 01 00 03 01 00 00 00 01 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00`

Key findings (see research-claude.md "Task 8: GTI OBDEleven $0C68 + RDID 006"):
- User GTI active functions (13): mFDR, ESH, MO, GE, VAQ, EPS, ACC, SAK, MO_StSt, AMB, KL, eBKV, AGK
- AFS = Not active (user has static LED headlights; ODIS-E reference GTI had adaptive headlights)
- RDID 006 bytes 12-23 match ODIS-E reference GTI exactly at the FPA-relevant positions
- 4 RDID 006 bits differ R vs GTI: byte 15 bit 0, byte 16 bit 5, byte 20 bit 1, byte 22 bit 1
  - byte 20 bit 1 = VAQ (confirmed), byte 22 bit 1 = ToS_L + ToS_Q (both gated together)
  - bytes 15/16 = ESP and ALR (order ambiguous without a one-but-not-other vehicle)

**Remaining gap**: VCDS full autoscan not obtained (OBDEleven used instead). The
`gw_longcoding_controls` cross-reference is complete for the FPA-relevant RDID 006 bits.
The "Byte N, bit M" labels in getLongCodingByteName() do NOT map to RDID 006 byte positions.

---

## File naming convention

Always include:
- Model (e.g. `golf8_gti`, `golf8_r`, `octavia_iv`)
- Gateway part number (e.g. `5WA907530J`)
- Dataset ID if known (e.g. `V03935345NU`)
- State / feature label if before/after pair

Example: `golf8_gti_5WA907530J_V03935345NU_baseline.bin`
