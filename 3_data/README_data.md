# Data

This folder contains the response data from both experiments.

---

## Contents

```
3_data/
├── README.md       ← You are here
├── exp1.csv        Response data (human + model, cloze-in-VWP)
└── exp2.csv        Attention data (human eye-tracking + model attention, standard VWP)
```

---

## exp1.csv — Response Data (Cloze-in-VWP)

Response data from the cloze-in-VWP task. After hearing the sentence ending with "but not…" (*而不是*), participants/model selected one of four images on screen and provided a text continuation. Contains data from both human participants and the multimodal LLM (Qwen2.5-Omni-7B).

### Basic statistics

| | Human | Qwen2.5-Omni-7B |
|---|---|---|
| Participants | 118 | 20 (Omni_01–Omni_20) |
| Items | 36 | 36 |
| Conditions | 6 (2 Structure × 3 Stress) | 6 |
| Rows | 4,246 | 4,320 |
| **Total** | **8,566 rows** | |

### Column descriptions

| Column | Type | Description |
|---|---|---|
| `Participant` | str | Participant type: `Human` or `Qwen` |
| `SubjectID` | str | Unique participant identifier (e.g., `007_zmm` for humans, `Omni_01` for model) |
| `List` | int | Latin-square list assignment (1–6) |
| `Item` | int | Item number (1–36) |
| `Response` | str | Raw selected quadrant: `TL`, `TR`, `BL`, or `BR` |
| `Response_update` | str | Corrected response after click repair (see below; differs from `Response` in 31 cases) |
| `Sentence` | str | Chinese dative sentence ending with "but not…" (*而不是……*) |
| `Continuation` | str | Text continuation provided by the participant/model after "but not" (*而不是*) |
| `Structure` | str | Sentence structure: `DO` (double-object) or `PO` (prepositional-object) |
| `Stress` | str | Prosodic condition: `Neutral`, `Recipient`, or `Theme` |
| `Audio_File_new` | str | Filename of the audio stimulus played |
| `RC_align` | 0/1 | **Response–Continuation alignment**: `1` if the selected image matches the text continuation; `0` if inconsistent |
| `New_pic` | 0/1 | **New picture selection**: `1` if the selected image is an alternative (unmentioned) entity; `0` if the participant clicked on an already-mentioned entity (mentioned recipient or mentioned theme) |
| `New_pic_updated` | 0/1 | Corrected version of `New_pic` after click repair |
| `R_con` | 0/1 | Continuation refers to the **alternative recipient** |
| `T_con` | 0/1 | Continuation refers to the **alternative theme** |
| `O_con` | 0/1 | Continuation is an **action, ambiguous, or off-task** response that does not clearly refer to any specific image entity |
| `No_im` | 0/1 | The continuation refers to an entity **not depicted** in any image, or **misnames** an image (e.g., naming a person as an object: soldier → "步枪/rifle", tourist → "相机/camera") |

### Response coding

**Continuation-based coding.** `R_con`, `T_con`, and `O_con` are mutually exclusive (exactly one equals 1 per row), coded based on what the text continuation refers to:
- `R_con = 1`: continuation refers to the alternative recipient (e.g., "护士/nurse", "农夫/farmer")
- `T_con = 1`: continuation refers to the alternative theme (e.g., "水果/fruit", "衬衫/shirt")
- `O_con = 1`: continuation is an action, ambiguous, or does not match any specific entity on screen (e.g., "自己带戒指离开/take the ring and leave himself", "给他拉小提琴/play the violin for him", "变了兔子出来/conjured a rabbit")

`No_im` is coded independently and indicates that the continuation refers to an entity not depicted in any image, or misnames an image (e.g., writing an object name for a person image: soldier → "步枪/rifle", tourist → "相机/camera").

**Data exclusion.** For analysis, rows with `O_con = 1` or `No_im = 1` were excluded, as these responses do not clearly map to any of the four image entities.

**Click repair procedure.** Some participants accidentally clicked on a mentioned (old) entity while their continuation clearly referred to an alternative (new) entity. In such cases:
1. `Response_update` was corrected to the quadrant containing the matching alternative entity.
2. `New_pic_updated` was changed from 0 to 1.
3. `RC_align` was updated accordingly.

Ambiguous continuations (e.g., "别人", "其他") were not repaired and retained the original click.

---

## exp2.csv — Attention Data (Standard VWP)

Time-course data from the standard VWP task. For humans, values are fixation proportions from eye-tracking; for the model, values are attention weights extracted from each layer of Qwen2.5-Omni-7B. Contains data from both human participants and the multimodal LLM.

### Column descriptions

| Column | Type | Description |
|---|---|---|
| `Participant` | str | Participant type: `Human` or `Model` |
| `Subject` | str | For humans: participant ID; for model: layer number (`L1`–`L28`) |
| `Item` | int | Item number (1–36) |
| `Time` | int | Time relative to onset of the target word (ms). Negative values = before target onset; positive values = after target onset. |
| `Stress` | str | Prosodic condition: `Neutral`, `Recipient`, or `Theme` |
| `Structure` | str | Sentence structure: `DO` or `PO` |
| `REST` | float | Fixation proportion (human) or attention weight (model) on **blank/non-image regions** |
| `T_altLoc_Prop` | float | Fixation proportion / attention weight on the **alternative theme** |
| `R_altLoc_Prop` | float | Fixation proportion / attention weight on the **alternative recipient** |
| `T_menLoc_Prop` | float | Fixation proportion / attention weight on the **mentioned theme** |
| `R_menLoc_Prop` | float | Fixation proportion / attention weight on the **mentioned recipient** |

### Temporal resolution

| | Human | Model |
|---|---|---|
| Sampling rate | 50 ms per bin | 40 ms per bin (Qwen2.5-Omni audio sampling rate = 25 Hz) |

Time reference for both: onset of the target word. Negative values = before target onset; positive values = after target onset.

The 50 ms bin size for human eye-tracking follows the standard convention in VWP research (e.g., Altmann & Kamide, 1999; Ito, Pickering & Corley, 2018; Corps, Liao & Pickering, 2023).

### Notes

- The five columns (`REST`, `T_altLoc_Prop`, `R_altLoc_Prop`, `T_menLoc_Prop`, `R_menLoc_Prop`) sum to 1 within each row, representing the full distribution of fixation (human) or attention (model) at each time point.
- **Dependent variable for statistical analysis**: log(*T_altLoc_Prop* / *R_altLoc_Prop*), i.e., the log-ratio of alternative theme to alternative recipient attention/fixation. Positive values indicate a bias toward the alternative theme; negative values indicate a bias toward the alternative recipient.
