<div align="center">


# CM-EVS:Sparse Panoramic RGB-D-Pose Data for Complete Scene Coverage


<p>
  <a href="https://github.com/Strange-animalss"><img src="https://img.shields.io/badge/GitHub-Strange--animalss-181717?logo=github&logoColor=white" alt="GitHub"></a>
  <a href="https://huggingface.co/anon-cmevs-2026"><img src="https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-anon--cmevs--2026-FFD21E" alt="Hugging Face"></a>
  <a href="https://www.aniverse.net.cn/"><img src="https://img.shields.io/badge/Website-aniverse.net.cn-2962FF" alt="Website"></a>
  <a href="#-license"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License: MIT"></a>
  <img src="https://img.shields.io/badge/python-3.10%2B-blue" alt="Python 3.10+">
</p>


<p><em>A scalable pipeline for curating panoramic RGB-D datasets — generate candidate viewpoints, greedily pick the smallest set that covers a 3D scene, and render high-quality ERP frames under a unified schema.</em></p>

</div>

---

## ✨ Highlights

- **🎯 Conflict-minimized view selection.** A greedy selector that trades raw coverage gain against pairwise viewpoint conflict, yielding fewer-but-better frames per scene.
- **🌐 Source-agnostic.** One unified ERP RGB + range-depth + pose schema across Blender indoor (`.blend`), HM3D / generic GLB, ScanNet++ PLY, and outdoor sources.
- **🔁 Reproducible end-to-end.** Single-command smoke test, dry-run, and full-scene runs — all metadata (candidates, per-step log, selected viewpoints) emitted to disk.
- **🤝 License-aware companion dataset.** Redistributable Blender indoor frames + adapter-only packages for restricted sources, MLCommons Croissant manifest included.
- **🧩 Modular core.** ERP projection, tangent-plane extraction, depth, and warping live in small, swappable modules under `core/`.

---

## 🔗 Links

<table>
<tr>
  <td>🐙 <b>GitHub organization</b></td>
  <td><a href="https://github.com/Strange-animalss">github.com/Strange-animalss</a></td>
</tr>
<tr>
  <td>🤗 <b>Hugging Face</b> (code &amp; datasets)</td>
  <td><a href="https://huggingface.co/anon-cmevs-2026">huggingface.co/anon-cmevs-2026</a></td>
</tr>
<tr>
  <td>🌐 <b>Company website</b></td>
  <td><a href="https://www.aniverse.net.cn/">aniverse.net.cn</a></td>
</tr>
</table>


---

## 📦 Pipeline Overview

```text
Blender indoor .blend scenes
   │
   ▼
 candidate generation  ──►  conflict-minimized view selection  ──►  selected ERP rendering
                                                                            │
                                                                            ▼
                                                    coverage  •  oracle-gap  •  quality audit
```

HM3D / GLB and ScanNet++ / PLY are supported as secondary adapters; the **Blender-indoor path is the recommended first route for reviewers**.

---

## 🚀 Review-Ready Entry Points

| Purpose                   | Command                                                      |
| :------------------------ | :----------------------------------------------------------- |
| 🧪 No-data smoke test      | `bash scripts/run_tiny.sh`                                   |
| 🔍 Blender-indoor dry run  | `DRY_RUN=1 BLENDER=/path/to/blender INPUT_DIR=/path/to/blend_scenes bash scripts/run_blender_indoor.sh` |
| 🏃 Full Blender-indoor run | `BLENDER=/path/to/blender INPUT_DIR=/path/to/blend_scenes bash scripts/run_blender_indoor.sh` |
| 📊 Summarize a run         | `python3 scripts/summarize_blender_indoor_run.py --output-root outputs/blender_indoor` |
| 🕵️ Anonymity check         | `bash scripts/check_anonymity.sh`                            |

> The smoke test runs without any private assets. The full Blender-indoor run requires local `.blend` scenes and a Blender executable.

---

## 🛠️ Installation

```bash
# Option A — Conda
conda env create -f environment.yml
conda activate cmevs

# Option B — venv + pip
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## ⚡ Minimal Smoke Test

```bash
bash scripts/run_tiny.sh
```

<details>
<summary><b>Expected outputs</b></summary>


```text
outputs/tiny/metadata/candidates.jsonl
outputs/tiny/metadata/selected_viewpoints.json
outputs/tiny/metadata/per_step_log.jsonl
outputs/tiny/renders/
outputs/tiny/results/coverage_main.csv
outputs/tiny/results/oracle_validation.csv
outputs/tiny/results/audit_50_frames.csv
```

</details>

This test validates the repository wiring and metadata contracts — it is **not** intended to reproduce paper-scale numbers.

---

## 🏗️ Primary Full Run: Blender Indoor

Put `.blend` scenes under `data/blender_indoor/`, or point `INPUT_DIR` to another directory. Nested layouts are supported; the first subdirectory under `INPUT_DIR` is used as the scene name.

**Dry run:**

```bash
DRY_RUN=1 \
BLENDER=/path/to/blender \
INPUT_DIR=data/blender_indoor \
OUTPUT_ROOT=outputs/blender_indoor \
bash scripts/run_blender_indoor.sh
```

**Full run:**

```bash
BLENDER=/path/to/blender \
INPUT_DIR=data/blender_indoor \
OUTPUT_ROOT=outputs/blender_indoor \
NUM_FRAMES=30 \
RESOLUTION=2048,1024 \
bash scripts/run_blender_indoor.sh
```

<details>
<summary><b>Equivalent direct CLI</b></summary>


```bash
export PYTHONPATH="$PWD:$PWD/pipelines:${PYTHONPATH:-}"

python3 pipelines/run_full_pipeline.py \
  --blender /path/to/blender \
  --input-dir data/blender_indoor \
  --output-root outputs/blender_indoor \
  --num-frames 30 \
  --resolution 2048,1024 \
  --grid-spacing 0.5 \
  --min-frames 5 \
  --stop-gain 0.08
```

</details>

---

## 🔌 Secondary Adapters

| Adapter         | Source format    | Config                         |
| :-------------- | :--------------- | :----------------------------- |
| Blender outdoor | `.glb` / `.gltf` | `configs/blender_outdoor.yaml` |
| HM3D            | `.glb` / `.gltf` | `configs/hm3d.yaml`            |
| ScanNet++       | `.ply`           | `configs/scannetpp.yaml`       |

These adapters are provided for completeness and robustness analyses; the Blender-indoor route remains the recommended first reviewer path.

---

## 📁 Repository Layout

```text
.
├── pipelines/          # full scene pipelines; Blender indoor is the primary path
├── scripts/            # review and reproduction entry points
├── configs/            # default and source-specific configs
├── core/               # ERP projection, tangent extraction, depth, warping
├── tools/              # semantic and navigability helpers
├── utils/              # IO and pose utilities
├── examples/           # tiny Blender-indoor-style metadata example
├── metadata_examples/  # JSON schemas for candidate/selection logs
├── data/               # local data mount point (not tracked)
├── third_party/        # optional external dependencies (not tracked)
└── results/            # generated result CSVs
```

---

## 📂 Data & Checkpoints

This repository does **not** redistribute third-party scene assets, dataset files, or model checkpoints. Put local assets under `data/` or pass absolute paths via CLI. The `data/` directory is gitignored.

**Optional — Depth Pro** for ERPT-style depth fusion:

```text
third_party/ml-depth-pro/
third_party/ml-depth-pro/checkpoints/depth_pro.pt
```

---

## 🤗 Companion Dataset

The CM-EVS dataset is released on Hugging Face alongside this code repository:

> 📂 **Dataset:** [`huggingface.co/datasets/anon-cmevs-2026/cmevs-erp-eval`](https://huggingface.co/datasets/anon-cmevs-2026/cmevs-erp-eval)
>
> 🏷️ **Namespace** (code & datasets): <https://huggingface.co/anon-cmevs-2026>

### What's in it

A coverage-curated panoramic RGB-D dataset built under the principle *maximize the geometric coverage of a 3D scene with the fewest ERP frames possible*.

| Source                 | License       | Released          | Scale                          |
| :--------------------- | :------------ | :---------------- | :----------------------------- |
| **Blender indoor**     | CC-BY 4.0     | 📦 Full RGB-D data | **374 scenes · 13,631 frames** |
| HM3D                   | upstream EULA | 🔧 Adapter only    | 401 rooms (regen)              |
| ScanNet++              | upstream ToS  | 🔧 Adapter only    | 500 scans (regen)              |
| OB3D (outdoor)         | upstream      | 🔧 Adapter only    | 24 (regen)                     |
| TartanGround (outdoor) | upstream      | 🔧 Adapter only    | 762 parts (regen)              |

### Unified schema

Each ERP frame ships as a triple under a single right-handed `+X` right / `+Y` up / `+Z` forward world frame (OpenCV camera convention):

| File                        | Format         | Description                                |
| :-------------------------- | :------------- | :----------------------------------------- |
| `panorama_{NNNN}.png`       | PNG, 2048×1024 | ERP RGB image                              |
| `panorama_{NNNN}_depth.npy` | float32        | Range depth (m); NaN/0 invalid             |
| `pose_{NNNN}.json`          | JSON           | scalar-first `q_wc = [w,x,y,z]` + position |

### Reviewer quick sample

The full Blender-indoor archive is ~109 GB. For reviewer-time inspection without a full download, [`sence_indoor_0001`](https://huggingface.co/datasets/anon-cmevs-2026/cmevs-erp-eval/tree/main/blender_indoor/scenes/sence_indoor_0001) (~250 MB; 33 panoramas + 33 depth arrays + 33 poses) is provided as a representative sample produced by the exact same end-to-end pipeline as the rest of the release.

The dataset card also includes the Datasheet (Gebru et al. 2021), integrity-verification commands (`shasum -a 256 -c SHA256SUMS`), and the MLCommons Croissant v1.0 manifest. The Croissant manifest under `dataset_metadata/croissant.json` in this repository mirrors the dataset's metadata at release time.

---

## 📊 Results

> 🚧 **Numbers below are placeholders.** The §6 evaluation experiments (fixed-budget coverage, oracle-gain validation, λ sweep, cross-source robustness, downstream depth) are still in flight. The final tables will be filled alongside the camera-ready paper release.

| Experiment              | Setting                                          | Primary metric        | Result |
| :---------------------- | :----------------------------------------------- | :-------------------- | :----: |
| Fixed-budget coverage   | 374-scene Blender indoor, *K* = 30 frames/scene  | Coverage @ *K*        | `TBD`  |
| Oracle-gain validation  | Greedy vs. oracle upper bound                    | Gain ratio            | `TBD`  |
| λ sweep                 | Conflict weight λ ∈ {0.0, …, 1.0}                | Coverage vs. conflict | `TBD`  |
| Cross-source robustness | Blender / HM3D / ScanNet++ / OB3D / TartanGround | Per-source coverage   | `TBD`  |
| Downstream depth        | Panoramic depth on 94-scene subset               | δ₁ / AbsRel           | `TBD`  |

---

## 🧪 Paper Experiments

The driver scripts for the §6 evaluation experiments — *fixed-budget coverage*, *oracle-gain validation*, *λ sweep*, *cross-source robustness*, *downstream depth* — are scheduled to be released alongside the camera-ready paper.

The current release ships:

- 🧠 **Algorithmic core** — `scripts/build_candidates.py`, `scripts/select_views.py`, `scripts/render_selected.py`
- 📐 **Per-stage evaluation building blocks** — `scripts/evaluate_coverage.py`, `scripts/evaluate_oracle_gap.py`, `scripts/audit_quality.py`
- 📋 **Metadata-contract example** — verifiable end-to-end via the smoke test above

---

## ✅ Final Submission Check

Before uploading the code URL or zip:

```bash
bash scripts/run_tiny.sh
rm -rf outputs
bash scripts/check_anonymity.sh
```

---

## 📜 License

Code is released under the **MIT License** for review. Dataset assets remain governed by their original licenses — see `LICENSE.md` in the dataset card for the per-component matrix.

---

<div align="center">
<sub>Maintained by the <a href="https://github.com/Strange-animalss">Strange-animalss</a> organization · <a href="https://www.aniverse.net.cn/">aniverse.net.cn</a></sub>
</div>

