# Attention Extraction & Region Analysis Pipeline

This pipeline extracts **audio-to-image cross-attention** from Qwen2.5-Omni-7B, then analyzes attention distribution across four image quadrants.

---

## Overview

The pipeline has two stages:

1. **Attention Extraction**: For each (audio, image) pair, run a forward pass through Qwen2.5-Omni-7B with `output_attentions=True`, extract per-layer attention matrices (audio tokens as Q, image tokens as K), and save them as `.pkl` files.
2. **Region Analysis**: Read `.pkl` files, map image tokens to four quadrant regions (TL/TR/BL/BR) based on pixel coordinates, aggregate attention per region, normalize, and export as `.csv` files.

## Requirements

- Python >= 3.10
- GPU with >= 40GB VRAM recommended (e.g., A100-40GB)
- Dependencies:
  ```
  pip install transformers>=4.57 torchcodec torchaudio openpyxl torch numpy pandas pillow huggingface_hub
  ```

## File Structure

```
2_attention_extraction/
├── README.md                              # This file
└── attention_extraction_pipeline.ipynb    # Main notebook
```

## How to Use

### Step 1: Prepare Your Excel File(s)

Place `.xlsx` files in your project root directory (`ROOT_DIR`). Each Excel file **must** contain the following columns:

| Column Name | Description | Example |
|---|---|---|
| `Item` | Unique identifier for each trial | `1`, `exp_01` |
| `Audio_File_new` | Path to the audio file (absolute or relative to audio dir) | `trial1.mp3`, `audio1.wav` |
| `Image_File` | Path to the image file (absolute or relative to image dir) | `scene1.jpg`, `img01.png` |

Notes:
- Relative paths are resolved against the configured audio/image directories.
- Absolute paths are used as-is.
- Additional columns in the Excel file are ignored.

### Step 2: Prepare Your Stimulus Files

- **Images**: Must be exactly **1008 × 756 pixels** (or match your `CANVAS_W` × `CANVAS_H` setting). The pipeline validates image dimensions strictly.
- **Audio**: Supported formats include `.mp3`, `.wav`, `.flac`, etc. (anything `torchaudio` can load). Audio is automatically resampled to 16 kHz.

### Step 3: Configure Paths in the Notebook

Open `attention_extraction_pipeline.ipynb` and edit the **User Configuration** cell (Cell 2):

```python
HF_HOME = Path("/path/to/hf_home")                     # HuggingFace cache
MODEL_DIR = Path("/path/to/hf_models/Qwen2.5-Omni-7B") # Model directory
ROOT_DIR = Path("/path/to/your/project_root")           # Project root with .xlsx files
BASE_IMAGE_DIR_STR = "/path/to/your/image_directory"    # Image stimuli directory
BASE_AUDIO_ROOT = "/path/to/your/audio_root"            # Audio root directory
```

### Step 4: Select Audio Mode

The pipeline supports three audio cut-off conditions. Set `AUDIO_MODE` in Cell 2:

```python
AUDIO_MODE = "before_tar"  # change this to switch conditions
```

| Mode | Audio Directory | Description |
|---|---|---|
| `"before_tar"` | `{BASE_AUDIO_ROOT}/audio_cut_before_tar` | Audio cut before the target word |
| `"before_er"` | `{BASE_AUDIO_ROOT}/audio_cut_before_er` | Audio cut before 而 |
| `"before_sp"` | `{BASE_AUDIO_ROOT}/audio_cut_before_sp` | Audio cut before the silence period |

Each mode automatically sets:
- **Audio input directory**: `{BASE_AUDIO_ROOT}/audio_cut_before_{tar|er|sp}`
- **PKL output**: `{ROOT_DIR}/audio_cut_before_{tar|er|sp}/attention_pkls_raw/`
- **CSV output**: `{ROOT_DIR}/audio_cut_before_{tar|er|sp}/quad_outputs_fixedboxes/`

To process a different condition, change `AUDIO_MODE` in Cell 2 and re-run from Cell 8 onward (no need to reload the model).

### Step 5: Configure Region Coordinates (if needed)

Default four quadrant regions (pixel coordinates `[x1, y1, x2, y2]`):

```python
REGIONS = {
    "TL": [84, 28, 392, 336],    # Top-Left
    "TR": [616, 28, 924, 336],   # Top-Right
    "BL": [84, 420, 392, 728],   # Bottom-Left
    "BR": [616, 420, 924, 728],  # Bottom-Right
}
```

Constraints:
- All coordinates must be multiples of 28 (= PATCH_SIZE × merge_size = 14 × 2).
- Regions must not overlap.
- Each region must fit within the canvas (1008 × 756).

### Step 6: Run the Notebook

Run cells sequentially:

| Cells | Stage | Description |
|---|---|---|
| 1–2 | Configuration | Set paths and `AUDIO_MODE` |
| 3–4 | Setup | Install dependencies and import libraries |
| 5–6 | Model | Download and load Qwen2.5-Omni-7B (~15 GB first run) |
| 7–8 | Extraction | Define functions and run attention extraction → `.pkl` files |
| 9–10 | Analysis | Define functions and run region analysis → `.csv` files |

To process another audio mode: edit Cell 2, re-run Cell 2, then re-run Cells 8 and 10.

## Output Files

### PKL Files (Attention Extraction)

Each `.pkl` file contains:
- `attn_per_layer`: List of `(Q_sub, K_sub)` numpy arrays (one per layer)
- `audio_indices`: Absolute token indices for audio tokens (Q axis)
- `image_indices`: Absolute token indices for image tokens (K axis)
- `eff_grid_thw`: Effective grid dimensions `(T, H_eff, W_eff)`
- `image_k_rel_eff`: Relative indices in the effective grid
- Various metadata for validation

### CSV Files (Region Analysis)

Two formats are exported for each PKL:

1. **Human-readable** (`*.human.csv`): Each layer column contains a string like `TL=0.25;TR=0.30;BL=0.20;BR=0.25`
2. **Machine-friendly** (`*.machine.csv`): Separate columns per layer per region, e.g., `L01_TL`, `L01_TR`, `L01_BL`, `L01_BR`, `L01_REST`

Both include:
- `q_idx`: Audio token index (0-based)
- `audio_abs_index`: Absolute position in the input sequence

## Advanced Settings

| Setting | Default | Description |
|---|---|---|
| `HEAD_AGG` | `"mean"` | Aggregation across attention heads: `"mean"` or `"max"` |
| `DTYPE_SAVE` | `"float16"` | Precision for saved attention values |
| `AUDIO_STRIDE` | `1` | Downsample audio tokens (1 = keep all) |
| `MAX_AUDIO_TOKENS` | `None` | Limit number of audio tokens (None = no limit) |
| `LAYER_STRIDE` | `1` | Downsample layers (1 = keep all) |
| `REGION_AGG` | `"mean"` | Region aggregation before normalization: `"mean"`, `"sum"`, or `"max"` |
| `INCLUDE_REST` | `True` | Include a REST region (non-quadrant image tokens) |
| `FAIL_FAST` | `True` | Stop batch on first error |

## Troubleshooting

- **"Model did not return attentions"**: Ensure the model is loaded with `attn_implementation="eager"`. Flash attention does not support `output_attentions=True`.
- **Out of memory**: Try increasing `AUDIO_STRIDE` or `LAYER_STRIDE`. Or use a GPU with more VRAM.
- **"Image resolution mismatch"**: All images must be exactly `CANVAS_W` × `CANVAS_H` pixels (default: 1008 × 756).
- **"Box boundary is not a multiple of 28"**: Region coordinates in `REGIONS` must all be multiples of 28.
- **PKL already exists**: Extraction skips items whose PKL already exists. Delete existing PKLs to re-extract.
