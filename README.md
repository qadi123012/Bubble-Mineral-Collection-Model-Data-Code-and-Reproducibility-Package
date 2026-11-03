# Bubble Mineral Collection Model: Data, Code and Reproducibility Package

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![Tested](https://img.shields.io/badge/tested-Python%203.12.3%20%7C%20Ubuntu%2024.04-brightgreen)
![Unit tests](https://img.shields.io/badge/unit%20tests-7%20passing-brightgreen)
![Pipeline](https://img.shields.io/badge/pipeline%20regeneration-identical%20to%206e--14-brightgreen)
![Status](https://img.shields.io/badge/status-pre--submission-orange)
<!-- Add after choosing a licence: ![License](https://img.shields.io/badge/license-TBD-lightgrey) -->
<!-- Add after archiving on Zenodo: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.XXXXXXX.svg)](https://doi.org/10.5281/zenodo.XXXXXXX) -->

Supplementary data, source code, calculations and audit material for the manuscript **"TITLE OF YOUR MANUSCRIPT"** (*Final_Manuscript_Cl.docx*). The work analyses one air bubble rising through a magnesite suspension (**system C**, 96 high-speed frames) and places the measurements in an **Ec x Ea x Es** collection-efficiency model for flotation.

Everything reported for system C can be regenerated from the raw frames with one command.

---

## Scope

This repository covers the single-bubble image analysis (system C) and the collection-efficiency calculations built on it. Results in pixel units come directly from the raw frames. Results in physical units are computed from the scale and frame rate set in `COMPUTATIONAL_MODELS/CONFIGURATION/parameters.json` (default 11.2 um/px and 2000 fps); change them there to recompute everything.

Details of every check: `AUDIT/FINAL_SCIENTIFIC_DATA_AND_REPRODUCIBILITY_AUDIT.pdf`.

---

## Table of contents
1. [Scientific background](#1-scientific-background)
2. [What this repository contains](#2-what-this-repository-contains)
3. [Repository structure](#3-repository-structure)
4. [Quick start](#4-quick-start)
5. [Reproducing specific results](#5-reproducing-specific-results)
6. [Using the collection model](#6-using-the-collection-model)
7. [Data description](#7-data-description)
8. [Verification summary](#8-verification-summary)
9. [Known issues and limitations](#9-known-issues-and-limitations)
10. [Data availability](#10-data-availability)
11. [Troubleshooting](#11-troubleshooting)
12. [How to cite](#12-how-to-cite)
13. [License](#13-license)
14. [Contributing and contact](#14-contributing-and-contact)

---

## 1. Scientific background

In froth flotation, a hydrophobic mineral particle is collected by a rising bubble when it (i) collides with the bubble, (ii) attaches, and (iii) stays attached. The overall efficiency is the product

```
Ecoll = Ec x Ea x Es
```

| Term | Meaning | Source in this work |
|---|---|---|
| Ec | collision efficiency | Yoon-Luttrell correlation, evaluated at the measured bubble Reynolds number |
| Ea | attachment efficiency | sliding-contact expression with induction time (input value 0.78) |
| Es | stability efficiency | scenario value 0.85 (range 0.80 to 0.95) |

The repository analyses a single rising bubble (equivalent diameter about 116 px, aspect ratio 1.09) with an image pipeline: bubble segmentation and tracking, detection of bright surface features (an image proxy for attached particles), PIV of the surrounding liquid, and an independent cross-check with Bubble Analyser 2.0. The results are placed beside the model as an illustration, not as a validation.

## 2. What this repository contains

- Raw frames and micrographs, author CSV outputs, and regenerated outputs
- The original image-analysis pipeline (unmodified) and a script that re-runs it from the raw frames
- A consolidated statistics script (216 statistics) and a claim-by-claim check of the manuscript (109 claims)
- An executable implementation of the collection model (Eqs. 1 to 12) with unit tests
- New analyses clearly labelled as such: frame-rate and scale sensitivity, induction times implied by the measured Ea, recovery of the smoothing parameters
- Supplementary Information (PDF and Word), an Excel workbook with live formulas, a data dictionary
- Figures with their source data
- A correction log, missing-information list, traceability table and full audit report

## 3. Repository structure

```
Research_Project/
|-- README.md
|-- run_all.sh                      one-command reproduction
|-- CHANGELOG.md
|-- REPRODUCIBILITY_CHECKLIST.md
|-- MANUSCRIPT/                     manuscript that was audited
|-- SUPPLEMENTARY_INFORMATION/      SI (PDF, DOCX, MD), Supplementary_Tables.xlsx, Figs S1-S3
|-- DATA/
|   |-- RAW_DATA/                   121 frames (864 x 496 JPG), 4 micrographs (2400 x 1600 PNG)
|   |-- PROCESSED_DATA/             author CSVs, CSVs with smoothing added, per-frame table, masks
|   |-- DERIVED_DATA/               recomputed statistics, sensitivity tables, regenerated author outputs,
|   |                               stale author JSON (kept for the record only)
|   `-- DATA_DICTIONARY.xlsx        every column: type, unit, definition, manuscript use
|-- COMPUTATIONAL_MODELS/
|   |-- single_bubble_collection_model/   collection_model.py + tests/
|   `-- CONFIGURATION/parameters.json     every scenario input with its verification status
|-- SOURCE_CODE/
|   |-- data_processing/            analysis_pipeline.py (authors, unmodified), regenerate_csvs_from_frames.py
|   |-- calculations/               statistics, sensitivity, workbook and dictionary builders
|   |-- plotting/                   supplementary figures, reproduced Figs 8-11
|   |-- validation/                 claim verification, smoothing recovery, audit-table builders
|   |-- legacy_as_supplied/         authors' scripts, untouched
|   `-- legacy_adapted/             same scripts, only path constants changed
|-- FIGURES/                        PNG/PDF figures and plotted source data
|-- AUDIT/                          audit report, correction log, missing information, test log
|-- DOCUMENTATION/                  installation, execution, methodology, validation, troubleshooting
`-- ENVIRONMENT/                    requirements.txt, environment.yml, software_versions.txt
```

## 4. Quick start

**Requirements:** Python 3.10 or newer (tested with 3.12.3), about 1 GB free disk, `git` only for the optional Bubble Analyser cross-check.

```bash
git clone https://github.com/<your-username>/<your-repository>.git
cd <your-repository>

python3 -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate
pip install -r ENVIRONMENT/requirements.txt

bash run_all.sh --with-regeneration  # about 1 minute
```

Expected console output:

```
7 tests passed
109 claims checked: {'PASS': 100, 'PASS*': 1, 'FAIL': 8}
ALL IDENTICAL
```

The 8 failures are discrepancies in the manuscript, not code errors (see section 9).

Conda alternative: `conda env create -f ENVIRONMENT/environment.yml && conda activate bubble-collection`.

## 5. Reproducing specific results

| Goal | Command | Output |
|---|---|---|
| Re-run the image pipeline on the raw frames and compare with the supplied CSVs | `python SOURCE_CODE/data_processing/regenerate_csvs_from_frames.py` | `ALL IDENTICAL` |
| Add the Bubble Analyser cross-check (needs internet and git) | `... regenerate_csvs_from_frames.py --with-bubble-analyser` | identical to 1.4e-14 |
| Recompute every statistic | `python SOURCE_CODE/calculations/compute_manuscript_statistics.py` | `DATA/DERIVED_DATA/manuscript_statistics.json` |
| Check manuscript numbers | `python SOURCE_CODE/validation/verify_manuscript_claims.py` | `AUDIT/claim_verification.csv` |
| Test other frame rates or scales | `python SOURCE_CODE/calculations/sensitivity_fps_scale.py` | `DATA/DERIVED_DATA/sensitivity_fps_scale.csv` |
| Regenerate figures | `python SOURCE_CODE/plotting/make_supplementary_figures.py` and `make_mass_and_gallery_figures.py` | `FIGURES/PUBLICATION_READY/` |
| Run the authors' original scripts | `bash SOURCE_CODE/legacy_adapted/run_all_legacy_adapted.sh` | `DATA/DERIVED_DATA/legacy_regenerated/` |

To change any assumption (frame rate, scale, fluid, particle size, Ea, Es), edit `COMPUTATIONAL_MODELS/CONFIGURATION/parameters.json`, or use the blue input cells on sheet `ST1_Inputs` of `SUPPLEMENTARY_INFORMATION/Supplementary_Tables.xlsx`, which recalculates live.

## 6. Using the collection model

```python
import sys
sys.path.insert(0, "COMPUTATIONAL_MODELS/single_bubble_collection_model")
import collection_model as cm

# bubble diameter 1.304 mm, rise velocity 34.8 mm/s (manuscript scenario)
result = cm.Scenario(d_b=1.304e-3, v_b=0.0348).evaluate()
print(result["Re"], result["Ec"], result["Ecoll"], result["Np"])   # 48.6, 2.9e-3, 1.9e-3, 1.16

# induction time implied by a measured attachment efficiency (Eq. 4 inverted)
cm.implied_induction_time(0.78, t_c=100.0)                          # 50.9 ms
```

Implemented: Re, Eo, We, Mo, Ec (Eq. 2), Ea (Eq. 4), induction-time surrogate (Eq. 7), adhesion and detachment forces and Es (Eq. 8), Np (Eq. 9), K (Eqs. 10 and 11), recovery R(t) (Eq. 12). Run the tests with `python COMPUTATIONAL_MODELS/single_bubble_collection_model/tests/test_collection_model.py`.

## 7. Data description

| Dataset | Size | Description |
|---|---|---|
| `RAW_DATA/system_C_single_bubble_frames/` | 121 JPG | Exported frames 0 to 120; frames 25 to 120 contain the bubble and are analysed (96 frames, 95 intervals) |
| `PROCESSED_DATA/supplied_csv/bubble_kinematics_and_particles.csv` | 96 x 28 | Bubble size, position, shape, feature statistics per frame |
| `.../individual_particles.csv` | 5,673 x 7 | Position and diameter of every detected surface feature |
| `.../piv_summary.csv` | 95 x 10 | Far-field PIV statistics per frame pair |
| `.../bubble_analyser_crosscheck.csv` | 96 x 5 | Independent diameter from Bubble Analyser 2.0 |
| `RAW_DATA/system_C_high_resolution_micrographs/` | 4 PNG | Micrographs IMG_9423, 9432, 9433, 9434 (no physical scale; not used for any reported number) |

All columns, units and definitions are in `DATA/DATA_DICTIONARY.xlsx`. Conventions: image y increases downward; SD is the population SD (n); smoothing is Savitzky-Golay with window 11 and order 3.

## 8. Verification summary

| Check | Result |
|---|---|
| Authors' pipeline re-run on raw frames vs supplied CSVs | identical (max difference 5.7e-14) |
| Bubble Analyser cross-check re-run | identical (1.4e-14) |
| Smoothing parameters recovered | window 11, order 3 reproduces published statistics to 5e-14 |
| Collection-model unit tests | 7 of 7 pass |
| Manuscript numbers recomputed from the data | 109 checked, 100 match, 1 matches under a stated definition, 8 listed in the correction log |
| Excel workbook | 11,830 formulas, 0 errors |
| Clean virtual-environment run | passed in 58 s |

Test details and what was not tested: `AUDIT/TEST_LOG.md`.

## 9. Notes

1. Physical-unit results (velocity, Re, Ec, Np, masses) scale with the frame rate and spatial scale in `parameters.json`; `DATA/DERIVED_DATA/sensitivity_fps_scale.csv` tabulates other values.
2. Corrections to individual printed numbers are listed in `AUDIT/CORRECTION_LOG.md`.
3. The detected surface features are image detections of bright objects on the bubble; the particle-mass figures assume spherical magnesite particles.
4. Figures 8 to 11 are rebuilt from the data; the layout is not pixel-identical to the manuscript.
5. `DATA/DERIVED_DATA/legacy_supplied_stale/` holds earlier-version outputs, superseded by `manuscript_statistics.json`.

## 10. Data availability

The raw frames and micrographs, processed and derived datasets and all code used for the single-bubble analysis are in this repository.

## 11. Troubleshooting

| Symptom | Fix |
|---|---|
| `ModuleNotFoundError: openpiv` | `pip install OpenPIV==0.26.1` |
| OpenCV import error on a headless server | `pip install opencv-python-headless` |
| Different PIV results or errors | use the tested versions in `ENVIRONMENT/requirements.txt` |
| Figures use a different font | `sudo apt install fonts-liberation` (cosmetic only) |
| Bubble Analyser step fails | run without `--with-bubble-analyser`; the supplied cross-check CSV is used anyway |

More: `DOCUMENTATION/TROUBLESHOOTING.md`.

## 12. How to cite

Replace the placeholders after publication and archiving (for example on Zenodo).

```bibtex
@misc{YOURKEY_dataset,
  author       = {Surname, Given Names and Surname, Given Names},
  title        = {Single-Bubble Mineral Collection Model: Data, Code and Reproducibility Package},
  year         = {2026},
  publisher    = {GitHub},
  howpublished = {\url{https://github.com/<your-username>/<your-repository>}},
  doi          = {10.5281/zenodo.XXXXXXX}
}
```

Please also cite the manuscript once it is published. Third-party tools used: OpenCV, OpenPIV, SciPy, NumPy, pandas, scikit-image, matplotlib, and Bubble Analyser 2.0 (https://github.com/dmesap/bubble_analyser; commit 63b22f3 was used for the cross-check).

## 13. License

**TO BE COMPLETED BY THE AUTHORS.** No licence has been chosen yet, so by default all rights are reserved and others may not reuse the material. Before making the repository public, add a `LICENSE` file. A common choice is MIT or Apache-2.0 for the code and CC BY 4.0 for the data and figures. The third-party Bubble Analyser repository has its own licence.

## 14. Contributing and contact

Issues and pull requests that fix errors, improve documentation or add tests are welcome. Please keep the unmodified author files (`SOURCE_CODE/data_processing/analysis_pipeline.py`, `SOURCE_CODE/legacy_as_supplied/`) unchanged and open an issue first for anything that would alter a reported number.

Corresponding author: **NAME, AFFILIATION, EMAIL** (to be completed).

## Acknowledgements

**TO BE COMPLETED** (funding, facilities, data providers).
