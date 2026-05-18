# Supplementary Materials

**Paper:** Predictive Processing in Humans and a Multimodal LLM: Structure–Prosody Integration in the Visual World Paradigm

---

## Directory Structure

```
supplementary_materials/
├── README.md                              ← You are here
├── 1_stimuli/
│   ├── README.md                          Detailed stimuli documentation
│   ├── stimuli_list.xlsx                  Master spreadsheet (36 items × 6 conditions)
│   ├── audio/                             216 MP3 files (complete sentences)
│   └── visual/                            Assembled 2×2 displays (1024×768)
└── 2_attention_extraction/
    ├── README.md                          Pipeline documentation
    └── attention_extraction_pipeline.ipynb
```

---

## 1. Stimuli (`1_stimuli/`)

Complete experimental materials for 36 items × 6 conditions (2 structures × 3 stress patterns = 216 trials). Includes:

- **Audio**: Complete sentence recordings (216 MP3 files) used in the multimodal LLM experiment. Audio for the cloze-in-VWP task (human experiment) is identical but truncated after "but not" (*而不是*), i.e., with the target word removed.
- **Visual**: Assembled 2×2 visual displays for the human experiment (1024 × 768 px). Model displays (1008 × 756 px) were regenerated programmatically from the same images. Individual referent images available upon request.

See `1_stimuli/README.md` for file naming conventions and design documentation.

## 2. Attention Extraction (`2_attention_extraction/`)

Jupyter notebook pipeline for extracting audio-to-image cross-attention from Qwen2.5-Omni-7B and analyzing attention distribution across four image quadrants. Supports three audio cut-off conditions (before target word, before 而, before silence period). See `2_attention_extraction/README.md` for configuration, usage, and output format.

## Note on Data Availability

Experimental data (response data and time-course attention/fixation data) are not included in the supplementary materials due to the 200 MB file size limit. Data will be made available upon publication.
