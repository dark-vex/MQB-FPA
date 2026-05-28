# ODX Files and FPA Dataset Decoding

**ODX files would be highly valuable** — they are essentially the "ground truth" that the `.bt` template has been reverse-engineering without access to.

---

## ODIS Engineering HTML Exports — Confirmed Findings (2026-05-14)

Source: VCTool 2.4.3.3 backup exports from two Golf VIII bench projects.

### Vehicle identity

| | Golf VIII GTI | Golf VIII R |
|---|---|---|
| VIN | WVWZZZCDZMW082253 | WVWZZZCDZNW170526 |
| SW part | 5WA907530J | 5WA907530Q |
| SW version | 7083 | 7099 |
| HW part | 5WA907530C | 5WA907530G |
| **ZDC (FPA dataset ID)** | **V03935345NU** | **V03935364D** |
| ZDC version | 0001 | 0001 |
| ODX | EV_GatewMQB2020 | EV_GatewMQB2020 |
| ODX version | 004008 | 005011 |

Both are **MQB2020 (Golf 8) gateways** — ODX identifier `EV_GatewMQB2020`, not the older `EV_GatewConti_*` family used on Golf 7 era. The `.bt` template was originally developed on Golf 7 era datasets; the FPA format is the same architecture and the control name table has since been fully updated for MQB2020.

### Gateway long coding

```
GTI:    00 00 80 8D 0F 00 00 00 00 00 00 00 01 03 03 32 53 01 60 09 02 00 01 00 03 01 00 00 00 01 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
Golf R: 00 00 80 8D 0F 00 00 00 00 00 00 00 01 03 03 33 73 01 60 09 00 00 03 00 03 01 00 00 00 01 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
```

Bytes 0–14 identical. Differences at bytes 15–22 encode the different module/function configuration (AWD, torque splitter, front LSD presence). These 48 bytes map to the `gw_longcoding_controls_with_links` / `gw_longcoding_controls_without_links` arrays at 0xA38 / 0xA56 in the FPA dataset.

### FPA_Funktion_* control names — complete list from ODIS

This is the exhaustive set of FPA control names as stored in ODIS adaptation channels. The `FPA_Funktion` prefix and on/off state per vehicle are as follows:

| ODIS name | GTI | Golf R | Interpretation |
|---|---|---|---|
| `FPA_Funktion_MO` | active | active | Engine management (Motorsteuerung) — control ID `0x01` in .bt |
| `FPA_Funktion_GE` | active | active | Gearbox DSG/S-Tronic — control ID `0x03` |
| `FPA_Funktion_EPS` | active | active | Electric power steering — control ID `0x05` |
| `FPA_Funktion_ACC` | active | active | Adaptive cruise control — control ID `0x09` |
| `FPA_Funktion_KL` | active | active | Climate control (Klimaanlage) — control ID `0x08` |
| `FPA_Funktion_MO_StSt` | active | active | Start/Stop — control ID `0x02` |
| `FPA_Funktion_SAK` | active | active | Interior sound actuator — control ID `0x0A` |
| `FPA_Funktion_AMB` | active | active | Ambient lighting |
| `FPA_Funktion_eBKV` | active | active | Electric brake booster (iBooster / elektrischer BKV) |
| `FPA_Funktion_AGK` | active | active | Active grille shutters (Aktivgitterklappe) |
| `FPA_Funktion_AFS` | active | active | Active front lighting / DLA headlights |
| `FPA_Funktion_mFDR` | active | active | Multi-function drive regulation (electronic diff / XDS) |
| `FPA_Funktion_ESH` | active | active | Soundaktor / Structure-borne sound actuator — control ID `0x1F` in .bt |
| `FPA_Funktion_VAQ` | **active** | not active | Vorderachsquersperre — front LSD (GTI exclusive) |
| `FPA_Funktion_ALR` | not active | **active** | Allrad — AWD coordination (Golf R exclusive) |
| `FPA_Funktion_ToS_L` | not active | **active** | Torque Splitter, longitudinal dynamics [Torque Splitter – Längsdynamik] (Golf R) |
| `FPA_Funktion_ToS_Q` | not active | **active** | Torque Splitter, lateral dynamics [Torque Splitter – Querdynamik] (Golf R) |
| `FPA_Funktion_ESP` | not active | **active** | ESC Sport/Off profile mode (Golf R only) |
| `FPA_Funktion_DR` | not active | not active | DCC adaptive dampers — control ID `0x07` (neither car has DCC) |
| `FPA_Funktion_AGA` | not active | not active | Active exhaust (Abgasgeräuschaktuator) — control ID `0x14` |
| `FPA_Funktion_EDS` | not active | not active | Electronic diff lock — control ID maps to long coding `0x19` |
| `FPA_Funktion_HHC` | not active | not active | Hill hold control |
| `FPA_Funktion_PDC` | not active | not active | Parking distance control |
| `FPA_Funktion_FMA` | not active | not active | Unknown |
| `FPA_Funktion_Freilauf` | not active | not active | Coasting / engine-off freewheeling |
| `FPA_Funktion_Freilauf_DefaultON` | not active | not active | Coasting, default-on variant |
| `FPA_Funktion_MO_BZS` | not active | not active | Engine cylinder deactivation (ACT) |
| `FPA_Funktion_RGS` | not active | not active | Unknown — possibly rear height / air suspension |
| `FPA_Funktion_IVB` | not active | not active | Unknown |
| `FPA_Funktion_HSP` | not active | not active | Unknown — possibly Hinterachslenkung Sport (rear axle steering premium) |
| `FPA_Funktion_RWB` | not active | not active | Unknown — possibly rear-wheel braking for torque vectoring |
| `FPA_Funktion_HDC` | not active | not active | Hill descent control |

### Additional adaptation parameters (DPS timing)

These tune FPA button debounce/timing behaviour — same values on both cars, likely dataset defaults:

| Parameter | Value |
|---|---|
| `Driving_Profile_Selection_Toogle_Time_Adaptation` | 500 ms |
| `Driving_Profile_Selection_Touch_Contact_Time_Adaption` | 2500 ms |
| `Driving_Profile_Selection_Indication_Control_Adaption` | 3000 ms |
| `Driving_Profile_Selection_Kombi_Offset_Adaption` | 0 |
| `Driving_Profile_Selection_Sync_Time_Adaptation` | 1500 ms |
| `Driving_Profile_Selection_Request_Repetition_Time_Adaptation` | 255000 ms |
| `Driving_Profile_Selection_Fiddle_Proofing_Time_Adaptation` | 10000 ms |
| `Driving_Profile_Selection_Fiddle_Proofing_Limit_Adaptation` | 100 |
| `Driving_Profile_Selection_Fiddle_Proofing_Lock_Adaptation` | 60000 ms |

### What is NOT in these files

The raw FPA dataset bytes (`V03935345NU` / `V03935364D`) are absent — VCTool exports adaptations but not the ZDC dataset payload. Obtaining the hex dump requires ODIS-E → ZDC → SET_FPA_MOD → read dataset.

---

## Wiki Cross-Reference — jilleb/MQB-FPA/wiki/dataset (2026-05-16)

Source: https://github.com/jilleb/MQB-FPA/wiki/dataset

### Dataset size

Wiki documents **4352 bytes** (0x1100) — this is the older MK7/Golf 7 format. The template already
handles this: `local int mk7 = (FileSize() == 0x1100 ? 1 : 0);`. Our MQB2020 Golf VIII datasets are
**4088 bytes** (0xFF8). Offsets in the wiki are for the 4352-byte format; the section order is the
same but absolute offsets differ.

### Control ID table — wiki vs template comparison

Full wiki control table confirmed against template `getControlName()`. Notable findings:

| Decimal | Hex | Wiki name | Template name | Status |
|---|---|---|---|---|
| 35 | 0x23 | Exhaust valves | Exhaustflap | Match ✓ (not unknown as I previously thought) |
| 44 | 0x2C | Hill Descent Assist | Hillstepdownassist | Match ✓ |
| 46 | 0x2E | **Parking assist** | Driverassist_2 (Range calc.) | Wiki more specific |
| 47 | 0x2F | Instrument cluster | Displaysetup_1 | Wiki more specific |
| 48 | 0x30 | Infotainment system | Displaysetup_2 | Wiki more specific |
| 51 | 0x33 | Electronic engine sound | Esound | Wiki more specific |
| 54 | 0x36 | ESC System | Ebkv (eBKV) | **Conflict** — template says electric brake booster |
| 76 | 0x4C | **Freilauf_DefaultON** | *(missing)* | Added to template |
| 77–79 | 0x4D–0x4F | *(not listed)* | MQB2020_unknown_4D/4E/4F | Confirmed new MQB2020 IDs |

For `0x36`: wiki says "ESC System" but template labels it eBKV (FPA_Funktion_eBKV = electric brake
booster, confirmed active in ODIS on both cars). The conflict is unresolved without ODX ground truth.

### Profile ID table

Wiki profile IDs match the template's existing values exactly (0x03=Dynamic/Sport, 0x05=Eco, 0x07=Individual). Our datasets' AID banner confirms this independently. No discrepancy.

---

## GTI vs Golf R Dataset Comparison (2026-05-15)

Source: `DA_0019_7208_5H0_2044_010FPA000003_MQB37WXSTANDARD.xml`  
Binary saved: `research-inputs/golf8_gti_5WA907530J_V03935345NU.bin`

| | Golf VIII GTI | Golf VIII R |
|---|---|---|
| Dataset version | `"2044"` | `"2033"` |
| Model year | 2021 | 2021 |
| Size | 4088 bytes | 4088 bytes |
| CRC32 (last 4 bytes) | `3D 13 20 AF` | `95 15 91 F0` |

**Only 13 non-header/non-CRC bytes differ.** Both datasets share identical:
- All header bytes (0x04–0x17) except the version string
- Group structure: 4 groups (0x21×4, 0x07×3, 0x22×2, 0x24×2)
- Profile layout: identical slots (Comfort/Normal/Sport/Offroad/Race at slots 0,2,3,4,6)
- `list_of_grouped_controls` (0x4CD): byte-for-byte identical
- `list_of_controls` (0x9EB): **byte-for-byte identical**

This confirms the "common template, variant activation" model: the dataset is the same for all MQB2020
Golf VIII variants; adaptation channels (FPA_Funktion_*) activate/deactivate individual controls per
vehicle configuration. The dataset does not embed vehicle-specific control lists.

### Where the 13 bytes differ

**1. group_children_request_values matrix (0x26–0x225)**

Row 2, col 2 — control 0x4D request value:

| | GTI | Golf R |
|---|---|---|
| Damper (0x07) request [col A] | 12 | 12 (same) |
| **0x4D request [col B]** | **6** | **3** |

Row 9 cols 11–14 also differ (GTI fill=6, Golf R fill=3) but these columns cover positions [22–29]
which are all `0x00` (None) in `list_of_controls` — vestigial fill values with no functional effect.

**2. control_is_allowed_to_change (0x296)**

| Control | GTI allowed in | Golf R allowed in |
|---|---|---|
| 0x15 at position [6] | Race, Normal | Sport, Normal |
| 0x15 at position [8] | Race, Normal | Sport, Normal |

On GTI: the differential control (VAQ) is adjustable in Race+Normal modes.
On Golf R: the equivalent (ALR/AWD) is adjustable in Sport+Normal modes.

**3. control_is_reset (0x2D2)**

| Control | GTI resets in | Golf R resets in |
|---|---|---|
| 0x15 at position [6] | Race, Sport | Offroad, Sport |
| 0x15 at position [8] | Race, Sport | Offroad, Sport |
| 0x23 at position [13] | Comfort | Normal |

**4. restart_value (0x30E)**

| Control | GTI default | Golf R default |
|---|---|---|
| 0x23 at position [13] | `0x01` | `0x02` |

### Revised control ID assessments

New control IDs (present in BOTH datasets — confirmed NOT Golf R exclusive):

| ID | Hex | Positions | Observations |
|---|---|---|---|
| 77 | `0x4D` | [5], [7] | Different group_children_request value GTI(6) vs R(3); all-zero in request_values matrix |
| 78 | `0x4E` | [17] | All-zero in request_values matrix; no behavioral diff between GTI and Golf R |
| 79 | `0x4F` | [3] | All-zero in request_values matrix; in Normal group alongside Gearbox/Engine/StartStop |
| 35 | `0x23` | [13] | **= Exhaustflap** (already in template, wiki-confirmed); restart default differs GTI=0x01 vs Golf R=0x02 |

Control 0x15 (`Frontaxlediff`) appears at positions [6] AND [8] with identical IDs but different
allowed/reset profiles per vehicle — same dataset position, different hardware activated per variant.

### AID banner and profile ID confirmation

`AID_profile_banner[12]` at 0x8C2 is **byte-for-byte identical** on GTI and Golf R:
`01 00 03 00 05 00 06 00 00 00 00 00`

| Slot | FPA profile ID | AID banner ID | AID display |
|---|---|---|---|
| 0 | 0x01 | 0x01 | Comfort |
| 2 | 0x03 | 0x03 | Sport |
| 3 | 0x04 | 0x00 | (not set — Offroad uses default display) |
| 4 | 0x05 | 0x05 | Eco |
| 6 | 0x07 | 0x06 | Individual |

**Corrects earlier interpretation**: profile IDs are global constants, not vehicle-specific. Both GTI and
Golf R have the same 5 profiles: Comfort, Sport, Offroad, Eco, Individual. Earlier notes claiming Golf R
has "Normal", "Sport", "Race" were incorrect.

The exhaust valve control (0x23) having different restart defaults (GTI=0x01/quiet, Golf R=0x02/louder)
is the only behavioural difference between GTI and Golf R outside of control 0x15 (differential)
per-profile availability.

### request_values[8][30] matrix

This 8×30 matrix at 0x8FB encodes what value is sent to each control module when a given profile is
selected. **Identical between GTI and Golf R.** Controls 0x4D, 0x4E, 0x4F all have all-zero rows here —
they do not send explicit request values to modules (may use binary on/off signalling instead).

---

## Golf R FPA Dataset Parse — V03935364D (2026-05-14)

Source: `0019_5H0959542J_..._V03935364D_*.xml` (ODIS `GetParametrizeData` response).  
Extracted: `DA_0019_7208_5H0_2033_010FPA000003_MQB37WXSTANDARD.xml` from base64 TAR inside XML.  
Binary saved: `research-inputs/golf8_r_5WA907530Q_V03935364D.bin`

### Header fields

| Field | Value | Notes |
|---|---|---|
| `dataset_version` (0x00) | `"2033"` | ASCII 4-byte version string |
| `header_04` | `0x64` | always 0x64 ✓ |
| `year` (0x05) | `0x0B` = 11 → **2021** | 11 + 2010 |
| `header_07` | `0x0A` | always 0x0A ✓ |
| `header_09` | `0x01` | not FF (non-Audi), not 0x00 (no eco button) |
| `header_0C` | `0x0F` | not Tiguan (0x0A) |
| Total size | 4088 bytes (`0xFF8`) | matches `<GROESSE-DEKOMPRIMIERT>0FF8</GROESSE-DEKOMPRIMIERT>` |

### Profile layout

| Slot | Profile ID | Name | Returns after restart |
|---|---|---|---|
| 0 | `0x01` | Comfort | Comfort |
| 2 | `0x03` | Sport | Sport |
| 3 | `0x04` | Offroad | Comfort (resets!) |
| 4 | `0x05` | Eco | Eco |
| 6 | `0x07` | Individual | Individual |

Profile IDs are global constants — both GTI and Golf R use the same 5 profiles (Comfort, Sport, Offroad, Eco, Individual). Earlier notes claiming Golf R uses "Normal/Sport/Race" labels were incorrect; see the GTI vs Golf R comparison section for the correction.

### Control groups (4 active groups)

| Group ID | Controls | Interpretation |
|---|---|---|
| `0x21` | 4 controls | Normal-mode group (DCC + others) |
| `0x07` | 3 controls | DCC/Damper-related group |
| `0x22` | 2 controls | Sport-mode group |
| `0x24` | 2 controls | DCC-mode group |

### list_of_grouped_controls active entries (0x4CD)

`21 07 22 05 09 0B 24 08 0E 0C 4E 1C 2C 17 ...`

Control IDs present: 0x05 (Steering), 0x07 (DCC/Damper), 0x08 (Climate), 0x09 (ACC), 0x0B (Steerablebeam/lights),
0x0C (Interior light), 0x0E (Belt pre-tensioner), 0x1C (Centre diff), 0x2C (Centre+rear diff), 0x17 (?),
0x4E (?78 — **new, not in template**).

### list_of_controls active entries (0x9EB)

`03 01 02 4F 07 4D 15 4D 15 05 09 0B 24 23 08 0E 0C 4E 1C 2C 17 ...`

Control IDs of note:
- `0x4D` (77) — appears twice; present in all datasets (GTI, Golf R, PHEV). Early guess of "ToS_L + ToS_Q" was wrong — ToS_L/ToS_Q are $0C68 bit-flag functions, not `list_of_controls` IDs. Still unidentified.
- `0x4F` (79) — appears once; takes the StartStop slot on both combustion and PHEV datasets. Still unidentified.
- `0x23` (35) — confirmed `Exhaustflap` (wiki + template); `ExhaustFlap` in `list_of_controls` is correct.
- `0x15` (21) — confirmed `Frontaxledifferential` in template; appears twice (positions [6] and [8]), addressing two distinct ECU channels or request sub-types.

### gw_longcoding section (0xA29)

The data at 0xA29–0xA6F encodes the gateway long coding control mapping. First block (0xA29, 30 bytes):
`31 FF 05 04 0F 1F 08 1E 09 1E 09 0C 0D 0A 0E 18 12 0B 10 17 02 16 06 00 ...`

Expected alignment with Golf R long coding bytes from VCTool:
`00 00 80 8D 0F 00 00 00 00 00 00 00 01 03 03 33 73 01 60 09 00 00 03 00 03 01 00 00 00 01 01 00 ...`

Cross-mapping between these two arrays would confirm the `getLongCodingByteName()` table. The ODIS-E ground truth export (task 4, now complete) provides the Golf R long coding with field labels — see "Golf R Gateway ODIS-E Ground Truth" section.

### PHEV block (0xAAA)

At `0xAAA`: `01 FF FF FF FF FF FF FF FF FF FF FF FF FF FF ...`  
The leading `0x01` is non-zero (template says "others: 00 32 33 34 00 00 00 01").  
This dataset is non-PHEV; the block is mostly `FF` padding. Confirmed it only carries meaningful
data on PHEV vehicle types.

### Checksum

Last 4 bytes `0xFF4–0xFF7`: `95 15 91 F0` — this is the dataset CRC32.

---

## What ODX Files Would Give You

ODX (Open Diagnostic Data Exchange, ISO 22901) files describe ECU diagnostic data at the parameter level. For the FPA dataset specifically they would provide:

- **Exact parameter names and descriptions** for every byte currently marked red (`unknown_*`) in the template
- **Valid value enumerations** — instead of `case 0x42: return "Unknown"` you'd have the actual VAG-defined label
- **Bit-level encoding rules** for the packed nibble fields
- **Dependencies and constraints** between parameters
- **PHEV block definitions** — that large `PHEV_unknown_*` section would be fully mapped
- **Checksum rules** (confirming the CRC32 at offset 4348/4084)

The template's own save script gives you the keys to look for:

```xml
DATEIID  = MQBCODING_FPA     ← the ODX container name
ZDC_NAME = SET_FPA_MOD       ← the parameterization job name
DIAGNOSTIC_ADDRESS = 0x0019  ← gateway ECU (J533)
LOGIN    = 20103             ← security access level
```

In an ODX file, `SET_FPA_MOD` would be a `DIAG-SERVICE` or `FUNCTIONAL-CLASS` entry containing every parameter definition for this dataset.

---

## Where to Find Them

### Legitimate paths (hard but possible)

| Source | Notes |
|--------|-------|
| **ODIS Engineering** (dealer tool) | Contains `.pdx` packages (PDX = packaged ODX). The FPA data lives in the J533 gateway ECU package. Requires a dealer/workshop subscription to ODIS-E. |
| **ODIS Service** | Consumer-facing version; less raw access but same underlying data. |
| **ElsaWin / ETKA** | VAG's workshop information system — doesn't expose raw ODX but has parameter documentation. |
| **VAG Group engineering portals** | Internal only; accessible to suppliers with development agreements. |

### Community/grey-area paths

| Source | Notes |
|--------|-------|
| **Ross-Tech VCDS forums** | The VCDS team has deep decoded knowledge of gateway coding. They don't distribute ODX but forum discussions often contain decoded field meanings that overlap with unknowns in this template. |
| **MQB/OBD11 community** | OBDeleven and community apps sometimes publish coding descriptions derived from ODX. |
| **Extracted ODIS databases** | Some community members have extracted and shared `.odx`/`.pdx` files from ODIS installations. These circulate in VAG enthusiast forums (VWWatercooled, Audizine, MQB Discord channels). Legally grey. |
| **AutoHex II / similar tools** | Some aftermarket diagnostic tools ship with partial ODX-derived databases. |

### What to search for

If trawling forums or file-sharing communities, the relevant identifiers are:

```
MQBCODING_FPA
SET_FPA_MOD
J533 ODX
3Q0907530 ODX
FPA_DS
```

File extensions to look for: `.odx`, `.pdx`, `.caf`, `.prg`

---

## How to Use ODX Files Once You Have Them

ODX is XML-based, so it's human-readable. Relevant tools:

```bash
# Parse with Python (pip install python-odx or pyodxtools)
# Or just grep the XML directly:
grep -i "SET_FPA_MOD" *.odx
grep -i "FPA" J533*.odx
```

The structure to look for inside the ODX:

```xml
<DIAG-SERVICE ID="SET_FPA_MOD">
  <PARAMS>
    <PARAM>
      <SHORT-NAME>PHEV_unknown_AA3</SHORT-NAME>  ← actual name here
      <BYTE-POSITION>2723</BYTE-POSITION>
      <CODED-TYPE>...</CODED-TYPE>
    </PARAM>
  </PARAMS>
</DIAG-SERVICE>
```

---

## Realistic Assessment

The `.bt` template's author has already reverse-engineered a large portion of what ODX would confirm — the core profile, control, and button sections are well-mapped. The remaining value from ODX access would be concentrated in:

1. The **PHEV block** (`0xAA3`–`0xF20`) — almost entirely unknown
2. The **button data unknowns** (`0x7F6`–`0x83A`)
3. The **`SettingBytes_again` / `SettingBytes_more`** fields at the end of the control section

For the well-understood sections, the template and the mqbtools.nl online parser are already sufficient for practical customization.

---

## Dataset Memory Map (Key Offsets)

| Offset | Field | Size | Description |
|--------|-------|------|-------------|
| `0x00` | `dataset_version` | 4 bytes | ASCII version, e.g. `J100`, `OC00` |
| `0x04` | `header_04` | 1 byte | Always `0x64` |
| `0x05` | `year` | 1 byte | Model year = value + 2010 |
| `0x06`–`0x0D` | `header1` remaining | 8 bytes | Various flags (eco button, Audi vs VW, etc.) |
| `0x0E`–`0x17` | `header2` | 10 bytes | More config flags |
| `0x26` | `group_children_request_values` | 32×15 bytes | Nibble-packed override values sent to control modules |
| `0x206` | `group_member_request_values_controls` | 8 bytes | Which controls are in each group |
| `0x20E` | `amount_of_members_per_group` | 8 bytes | Member count per group |
| `0x23E` | `members_in_group_[4]` | 16 bytes | Bitmask: which GW long-coding bits belong to each group |
| `0x296` | `control_is_allowed_to_change[30]` | 60 bytes | Per-control 16-bit bitmask: which profiles allow change |
| `0x2D2` | `control_is_reset[30]` | 60 bytes | Per-control 16-bit bitmask: remember setting after restart |
| `0x32C` | `restart_value[30]` | 30 bytes | Default setting per control after restart |
| `0x34D` | `FPA_profile[12]` | 12 bytes | Which profile button is at each position |
| `0x359` | `profile_returns_after_restart[12]` | 12 bytes | Profile to return to after car restart |
| `0x365` | `allow_return_to_profile` | 2 bytes | 16-bit flags per profile type |
| `0x383` | `profile[12]` | 12×30 = 360 bytes | Per-profile, per-control mode setting byte |
| `0x4D3` | `list_of_grouped_controls[30]` | 30 bytes | Control ID shown at each HMI position |
| `0x4F1` | `settings_shown_in_HMI[12]` | 12×60 = 720 bytes | Per-profile: 16-bit bitmask of selectable modes per control |
| `0x7BB` | button section | varies | Button types, behaviors, profile lists |
| `0x8C0` | feature flags | 2 bytes | `0x2E` = standard FPA enabled |
| `0x8C2` | `AID_profile_banner[12]` | 12 bytes | AID display banner ID per profile |
| `0x8F7` | `mode_light_on[12]` | 24 bytes | LED state per profile (0=off, 1=on, 2=blink) |
| `0x90F` | `request_values[8]` | 8×30 = 240 bytes | Nibble matrix: value sent to each module per profile |
| `0x9F8` | `list_of_controls[30]` | 30 bytes | Control ID per profile byte position |
| `0xA16` | `grouped_controls[30]` | 30 bytes | Group number per control |
| `0xA34` | `grouped_controls_controlbyte` | 4 bytes | Bitmask: how many controls are grouped |
| `0xA38` | `gw_longcoding_controls_with_links[30]` | 30 bytes | GW long coding byte+bit per control |
| `0xA56` | `gw_longcoding_controls_without_links[30]` | 30 bytes | Same, for linked controls |
| `0xAA3`+ | PHEV data | ~850 bytes | Hybrid/PHEV-specific configuration (largely unknown) |
| last 4 | `dataset_checksum` | 4 bytes | CRC32 of entire file minus last 4 bytes (Little Endian) |

---

## Programmatic Parsing

The example files (`Full Example for DCC Index B`, etc.) use comma-separated hex format. To parse:

```python
import struct, binascii

# 1. Convert comma-hex to binary
raw = open("Full Example for DCC Index B").read()
data = bytes(int(x, 16) for x in raw.split(","))

# 2. Read key fields
version  = data[0:4].decode('ascii')
year     = data[5] + 2010
profiles = list(data[0x34D:0x34D+12])   # FPA_profile[12]
restart  = list(data[0x359:0x359+12])   # profile_returns_after_restart
settings = [list(data[0x383 + i*30 : 0x383 + i*30 + 30]) for i in range(12)]
controls = list(data[0x4D3:0x4D3+30])   # list_of_grouped_controls
req_vals = [list(data[0x90F + i*30 : 0x90F + i*30 + 30]) for i in range(8)]
ctrl_list = list(data[0x9F8:0x9F8+30])  # list_of_controls

# 3. Verify checksum
expected   = struct.unpack_from('<I', data, len(data)-4)[0]
calculated = binascii.crc32(data[:-4]) & 0xFFFFFFFF
assert calculated == expected, f"Checksum mismatch: {calculated:#010x} != {expected:#010x}"

# 4. Decode nibble-packed request values
def decode_request_value(data, profile_pair_idx, control_idx):
    byte = data[0x90F + profile_pair_idx * 30 + control_idx]
    value_a = byte & 0x0F        # even profiles (0, 2, 4, 6)
    value_b = (byte >> 4) & 0x0F # odd profiles  (1, 3, 5, 7)
    return value_a, value_b
```

---

## Key Value Tables

### Profile IDs (`FPA_profile` / `getFPAName`)
| Value | Profile |
|-------|---------|
| `01` | Comfort |
| `02` | Auto/Normal |
| `03` | Dynamic/Sport |
| `04` | Offroad |
| `05` | Eco/Economy |
| `06` | Race |
| `07` | Individual |
| `08` | Range / Clubsport / Snow |
| `09` | Lift / PHEV 2 |
| `0F` | EV Off |

### Control IDs (`list_of_controls` / `getControlName`)
| ID | Internal Name | System |
|----|---------------|--------|
| `01` | Enginepower | Engine management |
| `02` | Start/Stop | ISG |
| `03` | Gearbox | DSG / S-Tronic |
| `05` | Steering assist | EPS |
| `07` | Damper | DCC |
| `08` | Climatecontrol | HVAC |
| `09` | Adaptivecruisecontrol | ACC |
| `0A` | Soundactuator | Interior sound |
| `0D` | Airsuspension | Air suspension |
| `14` | Active exhaust | Exhaust valves |
| `1C` | Brakecontrolsystem | ESC |
| `1D` | Rearaxlesteeringsystem | Rear axle steering |

### Mode settings (`profile[n].setting_byte` / `getSettingName`)
| Value | Mode |
|-------|------|
| `01` | Comfort |
| `02` | Normal |
| `03` | Sport |
| `04` | Offroad |
| `05` | Eco |
| `06` | Race |
| `07` | Individual / Eco+ |
| `08` | On |
| `09` | Off |
| `0A` | Snow |
| `0D` | Sand |

### Gateway Long Coding bytes (`getLongCodingByteName`)
| Value | Description |
|-------|-------------|
| `0x04` | Byte 12, bit 3 — MO (Engine Management) |
| `0x05` | Byte 12, bit 4 — GE (Transmission) |
| `0x08` | Byte 12, bit 7 — DR (DCC Damper Control) |
| `0x0C` | Byte 11, bit 3 — EPS (Power Steering) |
| `0x0D` | Byte 11, bit 4 — ACC |
| `0x0F` | Byte 11, bit 6 — MO_StSt (Start/Stop) |
| `0x12` | Byte 10, bit 1 — KL (Climate control) |
| `0x19` | Byte 09, bit 0 — EDS (Rear differential lock) |

---

## PHEV Dataset Parse — V03935350BG (2026-05-16)

Source file: `DA_0019_7208_5H0_2031_010FPA000003_MQB37WXPHEV.xml`
Extracted: `research-inputs/golf8_phev_V03935350BG.bin` — 4088 bytes, version `2031`
XML tag: `MQB37WXPHEV` — this is the PHEV variant gateway dataset.
Comparison base: Golf R dataset V03935364D (version 2033)
Total bytes differing from Golf R: **538** (vs only 13 between GTI and Golf R)

---

### Profile layout

| Slot | FPA_profile | Returns to | AID banner |
|------|------------|------------|------------|
| 0 | Comfort (0x01) | Comfort | Comfort (0x01) |
| 2 | Sport (0x03) | Sport | Sport (0x03) |
| 4 | Eco (0x05) | Eco | Eco (0x05) |
| 6 | Individual (0x07) | Individual | Individual (0x06) |

**Key finding**: PHEV has the same 4 standard FPA profiles as a non-PHEV Golf. The hybrid
drive mode selection (E-Mode / Battery Hold / Hybrid Auto / Battery Charge) is a separate
sub-system defined entirely within the PHEV block (0xAA3+), not via FPA_profile entries.

---

### list_of_controls comparison — PHEV vs GTI/Golf R

PHEV has 15 active controls (positions 0–14). GTI/Golf R have 21 (positions 0–20).

| Pos | GTI / Golf R | PHEV | Change |
|-----|-------------|------|--------|
| 0 | Gearbox (0x03) | Gearbox (0x03) | same |
| 1 | Engine (0x01) | Engine (0x01) | same |
| 2 | StartStop (0x02) | **0x4F** | Start/Stop absent; 0x4F moves here |
| 3 | 0x4F | DCC (0x07) | 0x4F displaced; DCC shifts up |
| 4 | DCC (0x07) | **0x4D** | 0x4D (first instance) moves here |
| 5 | 0x4D (first) | **0x4D** (first) | both have duplicate 0x4D here |
| 6 | FrontDiff (0x15) | Steering (0x05) | FrontDiff absent in PHEV |
| 7 | 0x4D (second) | ACC (0x09) | 0x4D second instance absent in PHEV |
| 8 | FrontDiff (0x15) second | MatrixLight (0x0B) | FrontDiff second instance absent |
| 9 | Steering (0x05) | SoundComponents (0x24) | |
| 10 | ACC (0x09) | Climate (0x08) | |
| 11 | MatrixLight (0x0B) | Pretensioner (0x0E) | |
| 12 | SoundComponents (0x24) | InteriorLight (0x0C) | |
| 13 | ExhaustFlap (0x23) | **0x4E** | ExhaustFlap absent; 0x4E takes slot |
| 14 | Climate (0x08) | CentreRearDiff (0x17) | **New**: AWD (electric rear motor) |
| 15 | Pretensioner (0x0E) | — | |
| 16 | InteriorLight (0x0C) | — | |
| 17 | 0x4E | — | 0x4E absent in PHEV (only at pos 13) |
| 18 | ESC (0x1C) | — | ESC absent in PHEV |
| 19 | HillDescentAssist (0x2C) | — | |
| 20 | CentreRearDiff (0x17) | — | Moved to pos 14 in PHEV |

**Notes on duplicate control IDs**: Both GTI/Golf R and PHEV have 0x4D listed at two
positions (`[5]` and `[7]` on GTI/Golf R; `[4]` and `[5]` on PHEV). In GTI/Golf R,
0x15 (FrontDiff) also appears twice (`[6]` and `[8]`). Duplicate entries appear
intentional — likely addressing two distinct ECU channels or request value sub-types
for the same physical control.

**Inferences about 0x4F**: Takes the StartStop position in the Normal group on both PHEV
and GTI/Golf R. Likely an MQB2020-era control that co-exists with (or partially replaces)
traditional Start/Stop management.

**Inferences about 0x4E**: In GTI/Golf R it sits at position 17 alongside ESC and
HillDescentAssist. In PHEV it takes the ExhaustFlap slot at position 13. This suggests
0x4E is an ESC/brake-related system that differs in configuration between combustion and
PHEV hardware.

---

### PHEV block — structured analysis (0xAA3–0xEFF)

#### Control registry (0xAA3–0xABB)

**PHEV_unknown_AA3[8]** (`00 32 34 35 36 37 38 01`):
PHEV-exclusive control IDs registered here: 0x32=EnergoACC, 0x34=Eabc, 0x35=Drivemode,
0x36=Ebkv, 0x37=Drivemode2, 0x38=Drivemode3. Six controls vs three (0x32, 0x33, 0x34)
on older/non-MQB2020 PHEV datasets. Control 0x33=Esound is skipped in this Golf VIII
PHEV dataset.

**PHEV_unknown_AAF[4]** (`03 03 03 FF`):
Template documents this as "always 0xFF" — but it has three `03` bytes on this PHEV.
The three active bytes likely correspond to three of the six PHEV-exclusive controls
that have a specific mode assignment (03 = Sport). The FF may mean "not applicable" for
the remaining three.

**PHEV_unknown_AB3+** (`01 03 02 03 03 02 FF...`):
Six sequential values (01, 03, 02, 03, 03, 02) matching the six registered PHEV control
IDs. These appear to be per-control default mode assignments: 01=Comfort, 02=Normal/Auto,
03=Sport. Pattern: EnergoACC→Comfort, Eabc→Sport, Drivemode→Normal, Ebkv→Sport,
Drivemode2→Sport, Drivemode3→Normal.

#### Threshold/configuration tables (0xBB0–0xBCF)

**PHEV_unknown_BB0[10]** (`01 01 01 01 01 01 00 00 00 00`):
Six `01` bytes (enabled) followed by four `00` bytes (disabled). Matches 6 PHEV control
slots active.

**PHEV_unknown_BBA[10] ushorts** (big-endian): `55, 47, 31, 62, 61, 59, 0, 0, 0, 0`
Six non-zero threshold values for the six PHEV hybrid operational slots. Candidates:
- Speed thresholds (km/h): 55 km/h is typical VW GTE EV-mode max speed
- SOC thresholds (%): 55%, 47%, 31%, 62%, 61%, 59%
- Power values (kW)

**PHEV_unknown_BCE[10]** (`4, 5, 6, 1, 2, 3, 255, 255, 255, 255`):
Mode indices 1–6 in rearranged order (4,5,6,1,2,3). Suggests internal mode slot assignment
for the 6 hybrid operational states.

#### BD8 mode-definition tables (0xBD8–0xC57)

Four 32-byte tables. Each contains triplet patterns followed by zeros:

**Table 1** (0xBD8): `[0C 06 1C] × 3, [0C 06 09] × 3, zeros`
Control-ID triplets. Values: 0x0C(InteriorLight), 0x06(ProgSteering), 0x1C(ESC) and
0x09(ACC). Likely defines which physical ECU controls are active for two groups of hybrid
operational modes (e.g., 3 modes with ESC-off, 3 modes with ACC).

**Table 2** (0xBF8): `[81 01 01] × 3, [01 01 01] × 6, zeros`
Flag bytes. 0x81 = bit 7 set (possibly "primary/master slot"), 0x01 = standard active.
First 3 entries are flagged differently from the remaining 6.

**Table 3** (0xC18): `[03 09 01], [03 09 03], [03 09 02], [03 08 04], [03 08 05], [03 08 06]`
Triplets mapping 6 operational slots to PHEV profile-mode pairs. The third byte cycles
through all 6 profile positions (1,3,2,4,5,6 matching PHEV slot order). The second byte
alternates between 0x09 (PHEV2) and 0x08 (PHEV1/Range). Pattern: slots 1-3 use PHEV mode
0x09, slots 4-6 use PHEV mode 0x08.

**Table 4** (0xC38): Same structure but with 0x0D (HybridCharge) and 0x0E (HybridArea)
instead of 0x09/0x08. Together Tables 3+4 enumerate 4 PHEV drive modes that each map to
all 6 profile slots.

#### Hybrid mode bitmask (0xC58–0xC6E)

**PHEV_unknown_C58[22]** contains values 0x01, 0x02, 0x04, 0x08 at positions [1],[7],[13],[21]
and the C6E region continues with 0x10, 0x20 — powers of 2 for bits 0–5. This is a
6-bit bitmask encoding the 6 operational PHEV hybrid modes.

#### Hybrid mode markers (0xDE4)

**PHEV_unknown_DE4[12]**: `6 5 1 4 1 3 2 1 1 6 5 1`
Template expects: `6 5 1 4 1 3 2 1 1 14 13 1` on older PHEV datasets.
First 9 values match exactly. Last 3 differ: this dataset has `6 5 1` while older datasets
have `14 13 1` (0x0E, 0x0D = HybridArea, HybridCharge profile IDs). The MQB2020 GTE
dataset cycles back to slot indices 6,5,1 rather than referring to dedicated hybrid profile
ID constants.

#### AID mode banners (PHEV block, 0xE9D)

**AID_mode_banner** in PHEV block (12 entries):
| Slot | Value | Meaning |
|------|-------|---------|
| 0 | 0x0A | E-Mode (EV only) |
| 1 | 0x0C | Battery Hold |
| 2 | 0x0B | Hybrid Auto |
| 3 | 0x0D | Battery Charge |
| 4–6 | 0x00 | not set |
| 7 | 0x0E | E-Mode currently not available |
| 8–9 | 0x15 | Hybrid Mode — unable to switch |
| 10–11 | 0x00 | not set |

**Conclusion**: The Golf VIII GTE has 4 active PHEV hybrid modes (E-Mode, Battery Hold,
Hybrid Auto, Battery Charge) plus unavailability states for AID banner display.

#### PHEV_unknown_CD9 = 0xDB

Template documents: 0=almost always, 0x80=Audi, 0x42=older PHEV (O2-TE series).
This dataset has 0xDB — a new value specific to MQB2020 PHEV generation. Bit pattern
`11011011` — could be a feature-flag bitmask for MQB2020-specific PHEV capabilities.

---

### Summary — what the PHEV block encodes

The PHEV block (0xAA3–0xEFF) defines a parallel control system for PHEV hybrid mode
management, layered on top of the standard FPA profile system:

1. **Registers 6 PHEV-exclusive control IDs** (0x32–0x38 range) used for EV/hybrid
   coordination that don't appear in `list_of_controls`
2. **Defines 4 PHEV hybrid drive modes** (E-Mode, Battery Hold, Hybrid Auto, Battery Charge)
   with AID banner assignments and availability state handling
3. **Maps hybrid modes to FPA profile slots** via BD8 tables — each of the 4 hybrid modes
   references all 6 FPA profile slot positions through triplet entries
4. **Sets per-control thresholds** (BBA ushorts: 55/47/31/62/61/59) and mode defaults
   (AB3+: assignments to Comfort/Normal/Sport per control)
5. **Bitmask register** (C58+) tracks which of the 6 hybrid operational states are active

---

## Golf R Gateway ODIS-E Ground Truth (2026-05-26)

**Source files:** `research-inputs/odis/GolfR.html` (ODIS-E full session export, 287 KB),
`research-inputs/odis/GolfGTI-Gateway.html`, `research-inputs/odis/GolfR-Gateway.html`
(older root-level VCTool exports), plus 6 ODIS-E screenshots.

### ECU identity

| Field | Value |
|---|---|
| VW/Audi part number | `5WA907530Q` |
| Hardware part number | `5WA907530G` |
| ASAM/ODX file | `EV_GatewMQB2020` |
| ASAM/ODX version | `005011` |

### Golf R 48-byte gateway long coding (RDID 006)

```
0000808D0F00000000000000010303337301600900000300030100000001010000
000000000000000000000000000000
```

Full hex bytes 0–47:

| Byte | Hex | Binary       | [LO]_ field group |
|------|-----|--------------|-------------------|
| 00   | 00  | 0000 0000    | EMHLR             |
| 01   | 00  | 0000 0000    | EMHLR             |
| 02   | 80  | 1000 0000    | EMHLR             |
| 03   | 8D  | 1000 1101    | EMHLR             |
| 04   | 0F  | 0000 1111    | EMHLR             |
| 05   | 00  | 0000 0000    | EMHLR             |
| 06   | 00  | 0000 0000    | EMHLR             |
| 07   | 00  | 0000 0000    | EMHLR             |
| 08   | 00  | 0000 0000    | EMHLR             |
| 09   | 00  | 0000 0000    | MVEM/HVEM         |
| 10   | 00  | 0000 0000    | MVEM/HVEM         |
| 11   | 00  | 0000 0000    | MVEM/HVEM         |
| 12   | 01  | 0000 0001    | MKE               |
| 13   | 03  | 0000 0011    | MKE / PaCo        |
| 14   | 03  | 0000 0011    | PaCo              |
| 15   | 33  | 0011 0011    | PaCo              |
| 16   | 73  | 0111 0011    | PaCo              |
| 17   | 01  | 0000 0001    | PaCo              |
| 18   | 60  | 0110 0000    | PaCo              |
| 19   | 09  | 0000 1001    | PaCo              |
| 20   | 00  | 0000 0000    | PaCo              |
| 21   | 00  | 0000 0000    | PaCo              |
| 22   | 03  | 0000 0011    | PaCo              |
| 23   | 00  | 0000 0000    | PaCo              |
| 24   | 03  | 0000 0011    | WUM / eASS        |
| 25   | 01  | 0000 0001    | BCmE              |
| 26   | 00  | 0000 0000    | RWB / EHR         |
| 27   | 00  | 0000 0000    | DS / HD           |
| 28   | 00  | 0000 0000    | IDS / SK3         |
| 29   | 01  | 0000 0001    | dev CAN flags     |
| 30   | 01  | 0000 0001    | dev CAN flags     |
| 31–47| 00  | 0000 0000    | (unused/reserved) |

Selected decoded [LO]_ fields (RDID 006):

| Field | Value |
|---|---|
| [LO]_MKE_Function | active |
| [LO]_MKE_Variante | 3 |
| [LO]_PaCo_Function | active |
| [LO]_PaCo_Traffic_side | Left_hand_traffic |
| [LO]_PaCo_Transmission | Automatic_Transmission |
| [LO]_PaCo_Drive_type | Combustion_engine |
| [LO]_PaCo_Trailer_hitch | without |
| [LO]_PaCo_Country | Europe |
| [LO]_WUM_Function | active |
| [LO]_BCmE_Function | active |
| [LO]_RWB_Function | active |
| [LO]_HD_Function | active |
| [LO]_IDS_Function | active |
| [LO]_MVEM_Function | not_active |
| [LO]_HVEM_Function | not_active |

### GTI-vs-Golf R coding diff (bytes that differ)

| Byte | GTI | Golf R | Delta |
|------|-----|--------|-------|
| 15   | 32  | 33     | +1 in low nibble |
| 16   | 53  | 73     | bit 5 added |
| 20   | 02  | 00     | 2 bits cleared |
| 22   | 01  | 03     | bit 1 added |

GTI full coding: `00 00 80 8D 0F 00 00 00 00 00 00 00 01 03 03 32 53 01 60 09 02 00 01 00 03 01 00 00 00 01 01 00 ...`

### $0C68 FPA_Funktion adaptation table (RDID 007)

The gateway $0C68 RDID lists 32 `FPA_Funktion_*` entries. The order is **not** 0x01–0x20
sequentially; it is 0x19–0x20 first, then 0x01–0x08, then 0x09–0x10, then 0x11–0x18.

| $0C68 pos | Template ID | Name | GTI | Golf R |
|-----------|-------------|------|-----|--------|
| 1  | 0x19 | FPA_Funktion_EDS | not active | not active |
| 2  | 0x1A | FPA_Funktion_HHC | not active | not active |
| 3  | 0x1B | FPA_Funktion_PDC | not active | not active |
| 4  | 0x1C | FPA_Funktion_FMA | not active | not active |
| 5  | 0x1D | FPA_Funktion_Freilauf_DefaultON | not active | not active |
| 6  | 0x1E | FPA_Funktion_mFDR | active | active |
| 7  | 0x1F | FPA_Funktion_ESH | active | active |
| 8  | 0x20 | FPA_Funktion_ToS_L | not active | **active** |
| 9  | 0x01 | FPA_Funktion_AGA | not active | not active |
| 10 | 0x02 | FPA_Funktion_ESP | not active | **active** |
| 11 | 0x03 | FPA_Funktion_Freilauf | not active | not active |
| 12 | 0x04 | FPA_Funktion_MO | active | active |
| 13 | 0x05 | FPA_Funktion_GE | active | active |
| 14 | 0x06 | FPA_Funktion_ALR | not active | **active** |
| 15 | 0x07 | FPA_Funktion_MO_BZS | not active | not active |
| 16 | 0x08 | FPA_Funktion_DR | not active | not active |
| 17 | 0x09 | FPA_Funktion_VAQ | **active** | not active |
| 18 | 0x0A | FPA_Funktion_AFS | active | active |
| 19 | 0x0B | FPA_Funktion_RGS | not active | not active |
| 20 | 0x0C | FPA_Funktion_EPS | active | active |
| 21 | 0x0D | FPA_Funktion_ACC | active | active |
| 22 | 0x0E | FPA_Funktion_SAK | active | active |
| 23 | 0x0F | FPA_Funktion_MO_StSt | active | active |
| 24 | 0x10 | FPA_Funktion_AMB | active | active |
| 25 | 0x11 | FPA_Funktion_IVB | not active | not active |
| 26 | 0x12 | FPA_Funktion_KL | active | active |
| 27 | 0x13 | FPA_Funktion_HSP | not active | not active |
| 28 | 0x14 | FPA_Funktion_ToS_Q | not active | **active** |
| 29 | 0x15 | FPA_Funktion_RWB | not active | not active |
| 30 | 0x16 | FPA_Funktion_HDC | not active | not active |
| 31 | 0x17 | FPA_Funktion_eBKV | active | active |
| 32 | 0x18 | FPA_Funktion_AGK | active | active |

GTI-vs-R $0C68 deltas (bolded above): Golf R activates ToS_L, ESP, ALR, ToS_Q; GTI activates VAQ.
This is consistent with Golf R = AWD + Torque Splitter; GTI = front-LSD (VAQ), no AWD.

### Template ID correction summary

The $0C68 order confirms these three template entries were wrong placeholders:

| Template ID | Old name | Confirmed ODIS name |
|-------------|----------|---------------------|
| 0x14 | `AV (Area View??)` | `FPA_Funktion_ToS_Q` (Torque Splitter, lateral dynamics [Torque Splitter – Querdynamik]) |
| 0x1F | `SchaltDR (Operating Mode Selection?)` | `FPA_Funktion_ESH` |
| 0x20 | `Stages DCC (??)` | `FPA_Funktion_ToS_L` (Torque Splitter, longitudinal dynamics [Torque Splitter – Längsdynamik]) |

29/32 names were already correct; these 3 are confirmed in the current `fpa_dataset.bt`.

### Open questions

1. **"Byte X bit Y" position annotations** — The `getLongCodingByteName()` prefixes (e.g.,
   "Byte 09, bit 6") do not match the actual 48-byte coding values. Golf R bytes 9–12 are all
   `00 00 00 01`, yet ~17 functions show as active. These annotations likely reference an internal
   sub-frame within $0C68 RDID 007, not the RDID 006 coding bytes. The two RDIDs are distinct
   objects. **The prefixes are left unchanged** until verified.

2. **`gw_longcoding` with/without links divergence** — `_with_links[16..23]` = `00 00 03 00 03 01 00 00`
   (Golf R), while `_without_links[17..29]` shows `03` at byte 22 for the R vs `01` for the GTI.
   The semantic meaning of the with/without split remains unverified.

3. **0x4D/0x4E/0x4F controls** — `$0C68` only covers IDs 0x01–0x20. The three unknown control IDs
   are **universal to all Golf VIII non-PHEV variants** — confirmed present and byte-for-byte identical
   in Golf 8 1.4 TSI (v2033), Golf R (v2033), and Golf 8 GTI (v2044). A basic variant cannot isolate
   them; ODIS-E parameter tree (Task 2) is the only remaining path to naming them.

---

## Four-Dataset Comparison — ClaudeDataset-investigation (2026-05-27)

Source: `ClaudeDataset-investigation/` folder, four renamed XML files.

| Filename | Model | Version | Dataset ID (inferred) |
|---|---|---|---|
| `…MQB37WXSTANDARD-Golf8-GTI.xml` | Golf 8 GTI | 2044 | V03935345NU |
| `…MQB37WXSTANDARD_Golf8-R.xml` | Golf 8 R | 2033 | V03935364D |
| `…MQB37WXSTANDARD_Golf8-14TSI.xml` | Golf 8 1.4 TSI (standard) | 2033 | V03935345ZH |
| `…MQB37WXPHEV-Golf8-14TSIe.xml` | Golf 8 GTE 1.4 TSIe PHEV | 2031 | V03935350BG |

### Key findings

**Golf 8 1.4 TSI is byte-for-byte identical to Golf R.** Both are version 2033, zero bytes differ.
This closes the hypothesis that a "standard variant without sports hardware" would have a different
`list_of_controls`. It does not.

**All three non-PHEV variants share an identical `list_of_controls`:**

```
[0] 0x03  Gearbox
[1] 0x01  Enginepower
[2] 0x02  Start/Stop
[3] 0x4F  ** UNKNOWN [4F] **
[4] 0x07  Damper (DCC)
[5] 0x4D  ** UNKNOWN [4D] **
[6] 0x15  Frontaxledifferential
[7] 0x4D  ** UNKNOWN [4D] **   ← intentional duplicate
[8] 0x15  Frontaxledifferential ← intentional duplicate
[9] 0x05  Steering assist
[10] 0x09 ACC
[11] 0x0B Steerablebeam
[12] 0x24 Soundcomponents
[13] 0x23 Exhaustflap
[14] 0x08 Climatecontrol
[15] 0x0E Pretensioner
[16] 0x0C Interiorlight (Ambient)
[17] 0x4E ** UNKNOWN [4E] **
[18] 0x1C Brakecontrolsystem
[19] 0x2C Hillstepdownassist
[20] 0x17 Centreandrearaxlediff (AWD)
```

### GTI vs Golf R — full 17-byte diff labelled

| Offset | GTI | R | Field |
|---|---|---|---|
| 0x0002–0x0003 | `34 34` | `33 33` | Version bytes ("2044" vs "2033") |
| 0x0046 | `6C` | `3C` | `group_children_request_values` row 2, byte 2 (request value nibbles) |
| 0x00B8–0x00BB | `06 06 06 06` | `03 03 03 03` | `group_children_request_values` row 9, bytes 11–14 (4 nibble pairs) |
| 0x02A3 | `22` | `06` | `control_is_allowed_to_change[6]` high byte → Frontaxlediff (0x15), profile bitmask |
| 0x02A7 | `22` | `06` | `control_is_allowed_to_change[8]` high byte → Frontaxlediff (0x15) dup, profile bitmask |
| 0x02DF | `44` | `0C` | `control_is_reset[6]` high byte → Frontaxlediff (0x15) |
| 0x02E3 | `44` | `0C` | `control_is_reset[8]` high byte → Frontaxlediff (0x15) dup |
| 0x02ED | `02` | `04` | `control_is_reset[13]` high byte → Exhaustflap (0x23) |
| 0x031B | `01` | `02` | `restart_value[13]` → Exhaustflap (0x23) default value after restart |
| 0x0FF4–0x0FF7 | `3D 13 20 AF` | `95 15 91 F0` | CRC/checksum |

**Interpretation:**
- The GTI has more aggressive front-diff settings: the Frontaxledifferential is allowed to change in a
  different set of profiles (GTI high-byte `0x22` = ecoplus+sport; R high-byte `0x06` = ecoplus+race),
  and uses a different reset-profile bitmask.
- The Exhaustflap default value and reset behavior differ: GTI uses mode `0x01`, Golf R uses mode `0x02`.
- The request-value rows (0x0046, 0x00B8–0x00BB) are likely DCC stiffness or front-diff torque values
  per profile, and the GTI has doubled values vs the Golf R (0x06 vs 0x03, 0x6C vs 0x3C).

### What the unknown IDs reveal

**None of the 17 differences touch 0x4D, 0x4E, or 0x4F.** Their `control_is_allowed_to_change`
and `restart_value` are identical between GTI and Golf R. No variant-specific tuning exists for them.

Additional observations from the `control_is_allowed_to_change` table:
- **0x4D** (positions 5 and 7): high byte = `0xFF` → allowed in **all profiles** (same as DCC 0x07)
- **0x4F** (position 3): high byte = `0x17` → allowed in notset, ecoplus, race, offroad
- **0x4E** (position 17): high byte = `0x07` → allowed in notset, ecoplus, race

These profile permission patterns do not correspond directly to any Golf VIII active profile names
(which are Comfort/Sport/Offroad/Eco/Individual). The bits reference a fixed global enum that includes
profiles unused in Golf VIII (race, ecoplus, etc.). They do not help identify the controls.

**Conclusion:** 0x4D, 0x4E, 0x4F are base-platform features present in every Golf VIII regardless of
optional hardware fitment. Their names can only be confirmed via the ODIS-E `SET_FPA_MOD` parameter
tree (Task 2).

---

## VCTool ODIS-E Format Exports — All Four Vehicles (2026-05-27)

Source: `ClaudeDataset-investigation/VCTools-Export-ODIS-Format/` — four VCTool HTML backups.
These are full-vehicle scans; each file contains data from multiple ECU modules.

### J533 Gateway identity confirmed in all four files

| Vehicle | J533 SW part | J533 HW part | ZDC (dataset ID) | ODX |
|---|---|---|---|---|
| Golf 8 GTI | 5WA907530J | 5WA907530C | V03935345NU | EV_GatewMQB2020 v004008 |
| Golf 8 R | 5WA907530Q | 5WA907530G | V03935364D | EV_GatewMQB2020 |
| Golf 8 1.4 TSI | **5WA907530J** | **5WA907530C** | V03935345ZH | EV_GatewMQB2020 |
| Golf 8 GTE PHEV | 5WA907530N | 5WA907530C | V03935350BG | EV_GatewMQB2020 |

Key finding: **14TSI J533 SW/HW is byte-for-byte the same part as the GTI** (`5WA907530J` / `5WA907530C`).
Only the ZDC (dataset) differs. Confirms that the dataset is the only per-variant customization layer.

### PHEV scan includes J666 (address 8107) — Antenna/Telematics Module

The PHEV full-scan captured a module absent from the other three exports:

| Field | Value |
|---|---|
| System | J666 at address 8107 — Antenna/Telematics Module |
| SW/HW | 5WA035741B |
| ZDC | V03935364CE |
| ODX | EV_TrxMLGEMQB37W v005006 |
| Coding | 02 38 00 11 04 00 80 |

The GTI and Golf R also have this module but it was not connected during those scans.
The 14TSI is a Chinese-market variant and does not have the telematics module at all.
`V03935364CE` is the telematics module's own ZDC dataset — **not FPA-relevant**; obtaining
it would not help identify unknown control IDs.

### FPA_Funktion_* comparison table — all four vehicles (from J533 in each)

| Function | GTI | Golf R | 14TSI | PHEV |
|---|---|---|---|---|
| FPA_Funktion | active | active | active | active |
| FPA_Personalisierung | active | active | active | active |
| FPA_BAP_Display | active | active | active | active |
| FPA_Kombi_Anzeige | active | active | active | active |
| FPA_Taster_Auswertung | Rising slope | Rising slope | Rising slope | Rising slope |
| FPA_BAP_Start_aktiv | active | active | active | active |
| FPA_Funktion_MO | active | active | active | active |
| FPA_Funktion_GE | active | active | active | active |
| FPA_Funktion_EPS | active | active | active | active |
| FPA_Funktion_ACC | active | active | active | active |
| FPA_Funktion_KL | active | active | active | active |
| FPA_Funktion_MO_StSt | active | active | active | active |
| FPA_Funktion_AMB | active | active | active | active |
| FPA_Funktion_eBKV | active | active | active | active |
| FPA_Funktion_mFDR | active | active | active | active |
| FPA_Funktion_ESH | active | active | active | active |
| FPA_Funktion_AFS | active | active | **not active** | active |
| FPA_Funktion_AGK | active | active | **not active** | **not active** |
| FPA_Funktion_SAK | active | active | **not active** | **not active** |
| FPA_Funktion_VAQ | active | not active | not active | not active |
| FPA_Funktion_ALR | not active | active | not active | not active |
| FPA_Funktion_ToS_L | not active | active | not active | not active |
| FPA_Funktion_ToS_Q | not active | active | not active | not active |
| FPA_Funktion_ESP | not active | active | not active | not active |
| FPA_Funktion_MO_BZS | not active | not active | not active | **active** |
| FPA_Funktion_DR | not active | not active | not active | not active |
| FPA_Funktion_AGA | not active | not active | not active | not active |
| FPA_Funktion_EDS | not active | not active | not active | not active |
| FPA_Funktion_HHC | not active | not active | not active | not active |
| FPA_Funktion_FMA | not active | not active | not active | not active |
| FPA_Funktion_Freilauf | not active | not active | not active | not active |
| FPA_Funktion_Freilauf_DefaultON | not active | not active | not active | not active |
| FPA_Funktion_RGS | not active | not active | not active | not active |
| FPA_Funktion_IVB | not active | not active | not active | not active |
| FPA_Funktion_HSP | not active | not active | not active | not active |
| FPA_Funktion_RWB | not active | not active | not active | not active |
| FPA_Funktion_HDC | not active | not active | not active | not active |
| FPA_Funktion_PDC | not active | not active | not active | not active |

### Key observations

**14TSI lacks three active functions** vs GTI (despite identical J533 HW/SW):
- `FPA_Funktion_AFS` = not active — no IQ.LIGHT / steerable headlights
- `FPA_Funktion_AGK` = not active — no active grille shutters (Aktivgitterklappe)
- `FPA_Funktion_SAK` = not active — no interior sound actuator (Soundaktor Kabine)

The J533 ZDC dataset is the same physical template (`5WA907530J` / `5WA907530C`) for GTI and 14TSI;
only the ZDC suffix (`NU` vs `ZH`) differs to reflect different FPA_Funktion_ activation patterns.

**PHEV gains one function** not seen on combustion variants:
- `FPA_Funktion_MO_BZS` = active — "Motor BZS" likely relates to hybrid engine-battery zone
  switching or hybrid drive mode management. Consistent with the PHEV's `Hybridcomponents (0x20)`
  control in the dataset.

**No new FPA_Funktion_ entries exist for 0x4D/0x4E/0x4F.** These three CharismaListFunctions
control IDs have no corresponding `FPA_Funktion_*` adaptation channel. They appear to be
automatically configured by the gateway rather than user-tunable adapter entries. The ODIS-E
`SET_FPA_MOD` parameter tree (Task 3) remains the only way to name them.

### DC_Kombination_FPA_DisplayConfig

8 entries per vehicle in the Digital Cockpit combination section, all `not active` on all four
vehicles. These configure whether the FPA view has a custom layout in the Digital Cockpit — not
used on these cars.

---

## Version 2056 Dataset — MQB37WXRXGOLF (2026-05-28)

### Source files

Six new dataset variants placed in `research-inputs/vctool/`:

| File | Version | Date | Notes |
|------|---------|------|-------|
| `DA_0019_7208_5H0_2056_010FPA000003_MQB37WXRXGOLF_REMEMBER-DRIVE-MODE.xml.txt` | 2056 | 2025-11-01 | Base RDM template |
| `DA_0019_7208_5H0_2056_010FPA000003_MQB37WXRXGOLF_REMEMBER-DRIVE-MODE_ALL-MODES-WITH-BUTTON-FOR...txt` | 2056 | 2025-11-01 | RDM button enabled |
| `DA_0019_7208_5H0_2056_010FPA000003_MQB37WXRXGOLF_REMEMBER-DRIVE-MODE_ALL-MODES-WITH-BUTTON-NO_...txt` | 2056 | 2025-11-01 | RDM button disabled |
| `RDM_Button_ESC_Exhaust_S_Plus.xml.txt` | — (custom "3003") | 2025-08-06 | ESC button ring + Exhaustflap + S Plus |
| `RDM_button_ESC-ON-Ring_20250608_123843_OE-FORMAT.xml.txt` | — (custom "3002") | 2025-06-08 | ESC ring set to ON position |
| `RDM_button_ESC-SPORT-Ring_20250608_123843_OE-FORMAT.xml.txt` | — (custom "3001") | 2025-06-08 | ESC ring set to SPORT position |

The three "RDM_button_ESC-*" and "RDM_Button_ESC_*" variants were generated with VCTool in OE-FORMAT
and received new version strings ("3001", "3002", "3003") assigned by VCTool rather than by VAG.
The base version 2056 is the OEM factory dataset for `MQB37WXRXGOLF`.

Also present in root: `0019_5H0959542J_5H0959542J_076_0022_WVWZZZCDZNW131097_V03935364D_a1bf55c1d252c3b03aa050c5b547e625.xml`
— an ODIS `GetParametrizeData` manifest for ZDC V03935364D (Golf R) listing all 6 parameter blocks.
The FPA block (START_ADDRESS 7208) is `DA_0019_7208_5H0_2033_010FPA000003_MQB37WXSTANDARD.xml`.

### New control IDs: 0x50 and 0x51

Version 2056 introduces **two new CharismaListFunctions IDs** not present in any of our
2019-era datasets (2031/2033/2044):

- **0x50** (80 decimal) — appears TWICE in list_of_controls (positions 5 and 8)
- **0x51** (81 decimal) — appears TWICE in list_of_controls (positions 6 and 9)

The `gw_longcoding_controls_with_links` table in the 2056 dataset links them as follows:

| Position | Control ID | LC byte | LC bit meaning |
|----------|-----------|---------|----------------|
| 4 | 0x4D (mFDR) | 0x05 | GE (Transmission Management) |
| 5 | 0x50 (NEW) | 0x04 | MO (Engine Management) |
| 6 | 0x51 (NEW) | 0x20 | ToS_L (Torque Splitter: longitudinal) |
| 7 | 0x4D (mFDR) | 0x08 | DR (Damper / DCC) |
| 8 | 0x50 (NEW) | 0x1D | Freewheeling Default ON |
| 9 | 0x51 (NEW) | 0x1F | ESH (Soundaktor) |

Each of 0x4D, 0x50, 0x51 appears twice with DIFFERENT gateway long-coding bit links — this is the
"two subchannels" pattern previously noted for 0x4D. Each occurrence controls a different
gateway aspect of the same functional group.

**0x50** is linked to Engine Management (MO) and Freewheeling Default ON — the engine/gearbox/sailing
domain. **0x51** is linked to ToS_L (Torque Splitter longitudinal dynamics) and ESH (Soundaktor).

### 0x51 is removed in ESC button ring variants

In the three ESC button ring configurations, both occurrences of 0x51 in `list_of_controls`
are zeroed out (0x51 → 0x00 at positions 6 and 9). The control disappears from the active
list when a physical ESC ring button is present.

**Interpretation**: 0x51 is the FPA soft-control for a feature that is superseded by
the physical ESC ring button. With a physical ring, the driver directly selects the ESC
mode (ON/SPORT/OFF); there is no need for the FPA menu to offer it. The ring variants
also carry "SPORT" mode for 0x51's associated functions (ToS_L / ESH), which aligns with
the ring providing direct Sport-Plus activation.

### Active control list comparison: 2033_R vs 2056_RDM

2033_R has 21 active controls; 2056_RDM has 18 active controls.

Controls **removed** from 2033_R that are absent in 2056_RDM:
- **0x02** (Start/Stop) — replaced by 0x4F (ESH) taking its slot
- **0x08** (Climate control) — not present
- **0x15** (Frontaxlediff, 0x15) — absent (AFS/SAK features not on this MQB37WXRXGOLF variant)
- **0x17** (AWD — Centreandrearaxlediff, 0x17) — absent
- **0x2C** (HDC — Hillstepdownassist, 0x2C) — absent
- **0x4E** (eBKV) — **removed entirely** from 2056; previously present in 2033_R/2044_GTI

Controls **added** in 2056_RDM:
- **0x50** × 2 — new (see above)
- **0x51** × 2 — new (see above)
- **0x1C** (Brakecontrol/ESC) — present in 2056 but not in 2033_R list_of_controls

Notable: eBKV (0x4E) disappears in 2056 and is replaced by 0x50/0x51 as the new
torque-splitter/engine-management control pair. The 2056 dataset is a newer Golf R variant
with the Remember Drive Mode feature (a 2022+ facelift capability).

### Controlled before/after diff analysis

All diffs are relative to the base 2056_RDM dataset. Offsets below are absolute byte positions.

**Group 1 — Button profile list changes (ALL variants vs base)**

Offsets 0x07E2, 0x07E6, 0x07EA map to `button_profile_lists[3]`, `[4]`, `[5]` first bytes.

In the base dataset:
- list[3]: `[Individual, None, None, None]`
- list[4]: `[None, None, None, None]`  
- list[5]: `[None, None, None, None]`

In all configured variants (RDM btn + ESC button):
- list[3].list_0: 7 → 9 (profile index changes)
- list[4].list_0: 0 → 8 (adds a profile to list 4)
- list[5].list_0: 0 → 7 (adds Individual to list 5)

The base dataset is an unconfigured template; all real-vehicle configurations re-wire the
button cycling lists, which is why this diff appears universally.

**Group 2 — RDM button enable/disable**

- 0x0803: `unknown_button_data_802[1]` = 0x43 (enabled) / 0x00 (disabled)
  - `RDM_btn_FOR` (button configured): 0x43
  - `RDM_btn_NO` (button not configured): 0x00
  This is the "Remember Drive Mode button" activation byte.

**Group 3 — ESC button ring: control_is_reset bitmask expansion**

When an ESC ring button is present, additional profiles are added to the reset-memory bitmask
for several controls. The affected entries in `control_is_reset[30]` (at 0x02D2):

| ctrl idx | list_of_controls value | Byte offset | Base | ESC variant | Interpretation |
|----------|----------------------|-------------|------|-------------|----------------|
| 0 | 0x03 (Gearbox) | 0x02D3 | 0x04 | 0x6C | Adds Sport+Race+Individual to reset memory |
| 1 | 0x01 (Enginepower) | 0x02D5 | 0x04 | 0x6C | Same |
| 5 | 0x50 (NEW) | 0x02DC | 0x08 | 0x0C | Adds Normal profile to reset memory |
| 6 | 0x51 (NEW) | 0x02DE | 0x08 | 0x0C | Same |
| 8 | 0x50 (NEW, 2nd) | 0x02E2 | 0x08 | 0x0C | Same |
| 9 | 0x51 (NEW, 2nd) | 0x02E4 | 0x08 | 0x0C | Same |
| 14 | 0x23 (Exhaustflap) | 0x02EF | 0x04 | 0x0E | Adds Sport+Offroad to reset memory |

Exhaustflap gains Sport+Offroad reset memory in ESC variants, consistent with the ring
enabling Sport mode exhaust behavior.

**Group 4 — ESC button ring: profile setting and HMI changes for control index 7 (Brakecontrol/ESC)**

`list_of_grouped_controls[7]` = 0x1C = Brakecontrol (ESC system).

The ESC mode setting changes across multiple data structures for this control:

- `profile[0].setting_byte[7]` (0x036C): Comfort profile ESC = 1 → 2 (Comfort → Normal)
- `profile[2].setting_byte[7]` (0x03A8): Offroad profile ESC = 3 → 2 (Sport → Normal)
- `profile[5].setting_byte[7]` (0x0402): profile[5] ESC = 6 → 2

Corresponding HMI setting bytes (`settings_shown_in_HMI[i].individual_setting_byte[7]`):
- 0x04FA, 0x0572, 0x0626, 0x0662: ESC HMI display bitmask changes

When a physical ESC button ring is present, the FPA's control over the ESC mode in individual
profiles is adjusted — some profiles revert to Normal ESC rather than Sport/Race mode
(because the ring handles those modes directly).

**Group 5 — ESC button ring: request value for mFDR in Sport**

- 0x0920: `request_values[1].request_value[7]`, request_value_B for control index 7
  - Base: 0x22 → Sport request B nibble = 2 (Normal)
  - ESC variants: 0x32 → Sport request B nibble = 3 (Sport)

  This means `list_of_grouped_controls[7]` = Brakecontrol in Sport profile gets request value 3
  instead of 2 when an ESC ring button is configured.

- 0x0955 (ESC_Exh_SPlus only): `request_values[3].request_value[0]`
  Changes Race profile request for control 0 (Drivetrain/Gearbox) from 0x06 to 0x0B.

- 0x09B6 (ESC_SPORT_Ring only): `request_values[6].request_value[7]`
  Changes profile 12/13 request value for control index 7 (Brakecontrol).

**Group 6 — ESC button ring: list_of_controls changes**

- 0x09F1: `list_of_controls[6]` = 0x51 → 0x00 (remove 0x51 from active list)
- 0x09F4: `list_of_controls[9]` = 0x51 → 0x00 (remove 0x51 from active list)

As noted above, 0x51 is deactivated when a physical ESC ring replaces the soft control.

### Summary of template offset coverage

The version 2056 before/after pairs have confirmed the following template regions:

| Region | Offset | Field | Status |
|--------|--------|-------|--------|
| Button profile cycling lists | 0x07D6 | `button_profile_lists[8]` — 8×4 bytes | CONFIRMED |
| RDM button config | 0x0803 | `unknown_button_data_802[1]` | CONFIRMED = RDM button enable |
| Profile ESC settings | 0x0365+j*30+7 | `profile[j].setting_byte[7]` | CONFIRMED = Brakecontrol per profile |
| HMI bitmask for ESC | 0x04EB+j*60+14/15 | `settings_shown_in_HMI[j].individual_setting_byte[7]` | CONFIRMED |
| Request values | 0x08FB + j*30 + i | `request_values[j].request_value[i]` | CONFIRMED |
| List of controls | 0x09EB | `list_of_controls[30]` | CONFIRMED (unchanged from previous) |

The `gw_longcoding_controls_with_links` table at 0xA45 in the 2056 dataset now provides
gateway long-coding bit assignments for 0x50 and 0x51, answering the open question about
these new IDs at a structural level. Their $0C68 function ID equivalents remain unconfirmed
but can be inferred from the LC bit mapping.

### 2056 active profile configuration

FPA_profile slots for MQB37WXRXGOLF:
- Slot 0 → Comfort (0x01)
- Slot 2 → Sport (0x03)
- Slot 5 → Race (0x06)
- Slot 6 → Individual (0x07)
- Slot 7 → 0x11 (Racetrack / Hybrid Charge Off)
- Slot 8 → 0x10 (Second Hold / Torque Vectoring)

Notably: **Normal** profile (0x02) is absent. This is a Golf R-specific configuration
where the profiles are Comfort / Sport / Race / Individual (no Normal). The 0x10 and 0x11
slots correspond to a non-standard profile numbering — likely "Sport Plus" and "Track" modes
specific to the 2056 RDM variant.

### Open questions

- What are the $0C68 FPA_Funktion_ ID equivalents for 0x50 and 0x51?
- What exact functional name does VAG give to 0x50 and 0x51 in the ODX schema?
- The 2056 dataset has no Normal profile — is this by design for "RX Golf" (Golf R extreme?)?
- Profile 0x10 and 0x11: what display names do these use in the HMI?

---

## $0C68 FPA_Funktion Adaptation Channels — Complete Ordered List (2026-05-28)

Source: ODIS-E HTML exports in `research-inputs/odis/GolfR-Gateway.html` and `GolfGTI-Gateway.html`, plus ODIS-E screenshots from 2026-05-26.

### Complete ordered $0C68 channel list

The `$0C68` RDID (Driving Profile Selection) returns 39 adaptation channels in order.
The first 7 entries are system-level; the remaining 32 are FPA_Funktion_* feature-enable channels.
The INDEX of each FPA_Funktion_* entry (0-based) is the value used in `gw_longcoding_controls_with_links`.

| $0C68 index | Channel name | Golf R | GTI |
|-------------|-------------|--------|-----|
| 0 | FPA_Funktion (master) | active | active |
| 1 | FPA_Personalisierung | active | active |
| 2 | FPA_Kombi_Status_Unterdruecken | not active | not active |
| 3 | FPA_BAP_Display | active | active |
| 4 | FPA_Kombi_Anzeige | active | active |
| 5 | FPA_Taster_Auswertung | Rising slope | Rising slope |
| 6 | FPA_BAP_Start_aktiv | active | active |
| 7 (0x07) | FPA_Funktion_EDS | not active | not active |
| 8 (0x08) | FPA_Funktion_HHC | not active | not active |
| 9 (0x09) | FPA_Funktion_PDC | not active | not active |
| 10 (0x0A) | FPA_Funktion_FMA | not active | not active |
| 11 (0x0B) | FPA_Funktion_Freilauf_DefaultON | not active | not active |
| 12 (0x0C) | FPA_Funktion_mFDR | active | active |
| 13 (0x0D) | FPA_Funktion_ESH | active | active |
| 14 (0x0E) | FPA_Funktion_ToS_L | **active** | not active |
| 15 (0x0F) | FPA_Funktion_AGA | not active | not active |
| 16 (0x10) | FPA_Funktion_ESP | **active** | not active |
| 17 (0x11) | FPA_Funktion_Freilauf | not active | not active |
| 18 (0x12) | FPA_Funktion_MO | active | active |
| 19 (0x13) | FPA_Funktion_GE | active | active |
| 20 (0x14) | FPA_Funktion_ALR | **active** | not active |
| 21 (0x15) | FPA_Funktion_MO_BZS | not active | not active |
| 22 (0x16) | FPA_Funktion_DR | not active | not active |
| 23 (0x17) | FPA_Funktion_VAQ | not active | **active** |
| 24 (0x18) | FPA_Funktion_AFS | active | active |
| 25 (0x19) | FPA_Funktion_RGS | not active | not active |
| 26 (0x1A) | FPA_Funktion_EPS | active | active |
| 27 (0x1B) | FPA_Funktion_ACC | active | active |
| 28 (0x1C) | FPA_Funktion_SAK | active | active |
| 29 (0x1D) | FPA_Funktion_MO_StSt | active | active |
| 30 (0x1E) | FPA_Funktion_AMB | active | active |
| 31 (0x1F) | FPA_Funktion_IVB | not active | not active |
| 32 (0x20) | FPA_Funktion_KL | active | active |
| 33 (0x21) | FPA_Funktion_HSP | not active | not active |
| 34 (0x22) | FPA_Funktion_ToS_Q | **active** | not active |
| 35 (0x23) | FPA_Funktion_RWB | not active | not active |
| 36 (0x24) | FPA_Funktion_HDC | not active | not active |
| 37 (0x25) | FPA_Funktion_eBKV | active | active |
| 38 (0x26) | FPA_Funktion_AGK | active | active |

**Bold** = differs between Golf R and GTI.

**NOTE:** The $0C68 channel index and the `gw_longcoding_controls_with_links` value are NOT
the same numbering. The LC link values (0x01–0x20) are J533 **long-coding bit positions**
(as described in the `getLongCodingByteName` template function), NOT $0C68 indices.

### $0C68 vs long-coding bit position mapping

The getLongCodingByteName lookup in fpa_dataset.bt maps LC bit values to J533 Byte N, Bit M:
- 0x01–0x08 → J533 long-coding Byte 12, bits 0–7 (AGA, ESP, Freilauf, MO, GE, ALR, MO_BZS, DR)
- 0x09–0x10 → J533 long-coding Byte 11, bits 0–7 (VAQ, AFS, RGS, EPS, ACC, SAK, MO_StSt, AMB)
- 0x11–0x18 → J533 long-coding Byte 10, bits 0–7 (IVB, KL, HSP, ToS_Q, RWB, HDC, eBKV, AGK)
- 0x19–0x20 → J533 long-coding Byte 9, bits 0–7 (EDS, HHC, PDC, FMA, Freilauf_DefaultON, mFDR, ESH, ToS_L)

The FPA writes these long-coding bits when changing drive mode — it configures the gateway LC
to tell other ECUs which operating mode to enter (e.g. Sport → engine management in Sport mode).

### CORRECTED gw_longcoding_controls_with_links for 2033 Golf R

(Previous session notes had the array layout wrong — 30+30 separate arrays, not 2 bytes per entry)

| Position | Control | with_link | without_link | Interpretation |
|----------|---------|-----------|--------------|----------------|
| [4] | 0x07 | GE | Freilauf | Suspension writes GE LC bit; Freilauf-linked |
| [5] | 0x4D | MO | Freilauf | mFDR instance 1 writes MO LC bit |
| [6] | 0x15 | MO_StSt | Freilauf | VAQ (first) writes MO_StSt LC bit |
| [7] | 0x4D | ToS_L | Freilauf | mFDR instance 2 writes ToS_L LC bit |
| [8] | 0x15 | DR | Freilauf | VAQ (second) writes DR LC bit |
| [9] | 0x05 | Freilauf_DefaultON | Freilauf | |
| [10] | 0x09 | VAQ | Freilauf | StSt writes VAQ LC bit |
| [11] | 0x0B | Freilauf_DefaultON | Freilauf | |
| [12] | 0x24 | VAQ | Freilauf | Soundcomponents writes VAQ LC bit |
| [13] | 0x23 | EPS | Freilauf | Exhaustflap writes EPS LC bit |
| [14] | 0x08 | ACC | Freilauf | |
| [15] | 0x0E | AFS | Freilauf | Ambient lights writes AFS LC bit |
| [16] | 0x0C | SAK | Freilauf | Climate writes SAK LC bit |
| [17] | 0x4E | AGK | Freilauf | eBKV_NEW writes AGK LC bit |
| [18] | 0x1C | KL | Freilauf | Brakecontrol/ESP writes KL LC bit |
| [19] | 0x2C | RGS | Freilauf | HDC writes RGS LC bit |
| [20] | 0x17 | AMB | Freilauf | StSt2 writes AMB LC bit |

Note: positions [0]–[3] = no LC link (0x00 = not_set); positions 21+ = residual non-zero values for unused slots.

### CORRECTED gw_longcoding for 2056 (new controls 0x50/0x51)

| Position | Control | with_link | without_link |
|----------|---------|-----------|--------------|
| [4] | 0x4D | GE | Freilauf | ← different from 2033 (was MO) |
| [5] | 0x50 | MO | Freilauf | ← 0x50 takes MO from 0x4D |
| [6] | 0x51 | ToS_L | Freilauf | ← 0x51 takes ToS_L from 0x4D |
| [7] | 0x4D | DR | Freilauf | ← different from 2033 (was ToS_L) |
| [8] | 0x50 | Freilauf_DefaultON | Freilauf | |
| [9] | 0x51 | ESH | Freilauf | ← 0x51 also writes ESH LC bit |
| [10] | 0x05 | mFDR | Freilauf | ← note: 2056 has new mFDR and ESH slots |
| [11] | 0x09 | Freilauf_DefaultON | Freilauf | |
| [12] | 0x0B | ESH | Freilauf | |
| [13] | 0x24 | mFDR | Freilauf | |

**Key insight**: Between 2033 and 2056, control 0x4D's LC bit assignments changed:
- 2033 0x4D: MO + ToS_L (engine management + torque splitter longitudinal)
- 2056 0x4D: GE + DR (transmission + damper/DCC)
- New 0x50 in 2056: MO + Freilauf_DefaultON (engine management + default sailing)
- New 0x51 in 2056: ToS_L + ESH (torque splitter longitudinal + soundaktor)

So 0x50 and 0x51 represent a SPLIT of functionality that was previously in 0x4D.
The ESC ring button removal of 0x51 makes sense: the physical ring directly handles
ToS_L and ESH (dynamic mode + sound), so the soft-control is no longer needed.


---

## J533 Gateway Long Coding — Golf R vs GTI Comparison (2026-05-28)

Raw 48-byte J533 RDID 006 coding values extracted from ODIS-E HTML exports:

```
Golf R: 0000808D0F00000000000000 01 03 03 33 73 01 60 09 00 00 03 00 03 01 00 00 00 01 01 00 ...00
GTI:    0000808D0F00000000000000 01 03 03 32 53 01 60 09 02 00 01 00 03 01 00 00 00 01 01 00 ...00
```

Differences at 4 bytes (0-indexed, confirmed identical elsewhere):

| Byte | Golf R | GTI   | Bit# | R   | GTI |
|------|--------|-------|------|-----|-----|
| 15   | 0x33   | 0x32  | 0    | 1   | 0   |
| 16   | 0x73   | 0x53  | 5    | 1   | 0   |
| 20   | 0x00   | 0x02  | 1    | 0   | 1   |
| 22   | 0x03   | 0x01  | 1    | 1   | 0   |

These 4 bit positions correspond to the 5 FPA_Funktion differences between Golf R and GTI:
- Golf R unique: FPA_Funktion_ToS_L, FPA_Funktion_ESP, FPA_Funktion_ALR, FPA_Funktion_ToS_Q
- GTI unique:    FPA_Funktion_VAQ

With 4 bits for 5 functions, one bit likely gates two related functions (e.g., ToS_L and ToS_Q
are both torque splitter, sharing the same hardware-presence bit).

### J533 long coding structure (RDID 006)

The 67 `[LO]_` parameters in the Golf R RDID 006 coding are:
- `[LO]_EMHLR_*` (20 params): battery/energy management
- `[LO]_MVEM_*` / `[LO]_HVEM_*` (6 params): mild/high-voltage hybrid (all not active on Golf R)
- `[LO]_MKE_*` (2 params): engine management (Variante=3 on Golf R)
- `[LO]_PaCo_*` (27 params): vehicle parameterization (transmission, drive type, driver assists, etc.)
- `[LO]_WUM_*` (2 params): wheel speed monitoring
- `[LO]_eASS_*`, `[LO]_BCmE_*`, `[LO]_RWB_*`, `[LO]_EHR_*`, `[LO]_DS_*`, `[LO]_HD_*`, `[LO]_IDS_*`, `[LO]_SK3_*`: various module enables

**KEY FINDING**: There are NO `[LO]_FPA_*` parameters in RDID 006. The FPA_Funktion
activation states are entirely in the `$0C68` adaptation (RDID 007). The J533 RDID 006
coding governs hardware configuration (hybrid, PaCo, subsystem enables), not FPA profiles.

Therefore, `gw_longcoding_controls_with_links` values (0x01–0x20) do **NOT** index into
the RDID 006 long coding. Their "Byte N, Bit M" labels in `getLongCodingByteName` are
a different address space — most likely an internal FPA service table or J533-specific
RDID, confirmed to be WRONG as J533 RDID 006 byte positions.

**What `gw_longcoding_controls_with_links` DO tell us:** the function-level names
(AGA, ESP, MO, GE, ToS_L, eBKV, AGK, etc.) are correct — each FPA control is associated
with a named J533 function. The exact byte/bit position within J533 firmware remains
unresolved without the ODX schema or a matched VCDS scan (Task 8).


---

## Cross-Dataset gw_longcoding Summary (all 4 versions, 2026-05-28)

Full comparison of gw_longcoding_controls_with_links across all known dataset versions.

| Dataset | Version | Ctl 0x4D inst.1 | Ctl 0x4D inst.2 | Ctl 0x4E | Ctl 0x4F | Ctl 0x50 | Ctl 0x51 |
|---------|---------|----------------|----------------|----------|----------|----------|----------|
| 2031 PHEV | V03935350BG | GE (0x05) at pos[4] | MO (0x04) at pos[5] | SAK (0x0E) at pos[13] | not_set | absent | absent |
| 2033 Golf R | V03935364D | MO (0x04) at pos[5] | ToS_L (0x20) at pos[7] | AGK (0x18) at pos[17] | not_set | absent | absent |
| 2044 GTI | V03935345NU | MO (0x04) at pos[5] | ToS_L (0x20) at pos[7] | AGK (0x18) at pos[17] | not_set | absent | absent |
| 2056 RDM | MQB37WXRXGOLF | GE (0x05) at pos[4] | DR (0x08) at pos[7] | absent | not_set | MO+Freilauf_DefaultON | ToS_L+ESH |

Key observations:
- 0x4D's LC links CHANGE across versions/variants: each instance maps to the hardware subsystems relevant for that vehicle
  - PHEV: coordinates GE (transmission) + MO (engine) — hybrid drive management
  - Golf R/GTI 2033/2044: coordinates MO (engine) + ToS_L (torque splitter) — AWD performance
  - Golf R 2056 "RX": coordinates GE (transmission) + DR (damper/DCC) — newer architecture
- 0x4E's LC link also changes: SAK on PHEV vs AGK on combustion cars
  - PHEV: 0x4E writes SAK (interior sound actuator) — no exhaust flap on PHEV
  - Golf R/GTI: 0x4E writes AGK (exhaust valve) — active exhaust present
- 0x4F always has not_set (no LC gate) across all versions
- 0x50/0x51 only in 2056, taking over the MO+ToS_L functions from 0x4D

This confirms 0x4D is a GENERIC "drive dynamics coordinator" whose LC assignments
are dataset-specific based on what hardware is installed. Its ODX name likely
describes the coordination role rather than any specific subsystem.


---

## GTI OBDEleven Module Inventory (2026-05-28)

Source: `ClaudeDataset-investigation/OBDeleven_Log.txt`
Car: Golf 8 GTI "Daniel-San", VIN redacted, 2021, ~40493 km, scan date 2026-05-28

### GTI J533 gateway identification
- Module 19: GW2020 High, SW 5WA907530**N** v7312, HW 5WA907530C v752
- Note: Golf R has 5WA907530**Q** (different SW variant)

### Full module list (24 ECUs)

| Addr | Description | System | SW Part | Notes |
|------|-------------|--------|---------|-------|
| 01 | Engine | R4 2.0l TFSI | 8Y0906264 | 2.0 TSI, DNPA engine code |
| 02 | Transmission | GSG DQ381 | 0GC906557B | 7-speed DSG, dual-clutch |
| 03 | Brakes | ESC | 5WA614517BL | ABS/ESC module |
| 08 | Air Conditioning | Climatronic | 5WA907727BB | |
| 09 | Central Electrics | BCM 37W BOSCH | 5WA937086E | |
| 13 | Adaptive Cruise Control | ACC Bosch MQB | 5WA907572A | FPA: ACC active |
| 15 | Airbag | AirbagVW40 | 5WA959655H | |
| 17 | Dashboard | KOMBI | 5H0920340A | |
| 19 | Gateway | GW2020 High | 5WA907530N | FPA ECU, ZDC V03935345NU |
| 2B | Steering Column Lock | ELV-MQBB | 2Q0905861B | |
| 32 | Lock Electronics | **Quersperre** | 5WA907554B | **VAQ** — front LSD! |
| 42 | Driver Door | TSG FS | 5Q0959593K | |
| 44 | Steering Assistance | BASGEN1MQB37 | 5WA907145G | EPS |
| 52 | Passenger Door | TSG BFS | 5Q0959592K | |
| 5F | Multimedia | MU-O-ND-EU | 5H0035816K | |
| 6C | Rear View Camera | RV eCompact | 5WA980556B | |
| 75 | Telematics | OCU3HMQB37W | 5WA035284J | |
| 76 | Parking Assistant | PDC 08 Kanal | 5WA919294C | |
| 8107 | Antenna module | TrxModulHigh | 5WA035741B | |
| A5 | Driver Assistance | MQB MFK 3.0 | 5WA980653A | ADAS camera |
| A9 | Structure Borne Sound | **SAS-GEN 2.5** | 5H0907159 | **ESH** — soundaktor |
| B7 | Start System Interface | Kessy IOBOX | 5WA959436B | |
| D6/D7 | LED Modules L/R | LED1L/LED1R | 992941571AE | adaptive headlights (AFS) |

### FPA-relevant module cross-reference

| FPA_Funktion | Active? | Module | Confirms |
|-------------|---------|--------|---------|
| FPA_Funktion_VAQ | **yes (GTI)** | 32 "Quersperre" | VAQ ECU present |
| FPA_Funktion_ESH | yes | A9 "Structure Borne Sound" | ESH/soundaktor present |
| FPA_Funktion_ACC | yes | 13 "Adaptive Cruise Control" | ACC ECU present |
| FPA_Funktion_AFS | yes | D6/D7 LED modules | Adaptive headlights present |
| FPA_Funktion_EPS | yes | 44 "Steering Assistance" | EPS present |
| FPA_Funktion_ToS_L | **no (GTI)** | (absent) | No torque splitter — Golf R has 8126/8127 |
| FPA_Funktion_ToS_Q | **no (GTI)** | (absent) | Same — GTI no rear AWD |
| FPA_Funktion_ALR | **no (GTI)** | (absent) | No rear LSD |

Module 32 "Quersperre" (literally: transverse/cross lock = front differential lock) is the
VAQ ECU, unique to GTI. This module is absent on the Golf R, which instead has modules
8126 and 8127 (rear torque splitter, left/right axle) for FPA_Funktion_ToS_L/ToS_Q.

### What this file does NOT contain (needed for Task 8)

The OBDEleven fault scan does not include:
1. J533 48-byte long coding (RDID 006) — needed to map gw_longcoding bit positions
2. $0C68 FPA_Funktion adaptation channel values
3. Any per-module long coding/adaptation

To complete Task 8, read from OBDEleven → Module 19 (Gateway):
- "Coding" or "Long Coding" → copy the raw hex string (48 bytes = 96 hex chars)
- "Adaptations" → filter for "Driving profile" to see $0C68 FPA_Funktion channels

