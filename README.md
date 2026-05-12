# CM-EVS

This repository contains the code release for **CM-EVS: Conflict-Minimized Efficient View Selection for Scalable 3D Scene Data Acquisition**.

- **Company website:** <https://www.aniverse.net.cn/>
- **GitHub organization:** <https://github.com/Strange-animalss>
- **Hugging Face:** <https://huggingface.co/anon-cmevs-2026> (code and datasets)

The release is intentionally organized around one primary review path:

```text
Blender indoor .blend scenes
  -> candidate generation
  -> conflict-minimized view selection
  -> selected ERP rendering
  -> coverage, oracle-gap, and quality-audit outputs
```

HM3D/GLB and ScanNet++/PLY support is included as secondary adapters, but the first path reviewers should inspect is the Blender-indoor path.

## Review-Ready Entry Points

| Purpose                      | Command                                                      |
| ---------------------------- | ------------------------------------------------------------ |
| No-data smoke test           | `bash scripts/run_tiny.sh`                                   |
| Blender-indoor dry run       | `DRY_RUN=1 BLENDER=/path/to/blender INPUT_DIR=/path/to/blend_scenes bash scripts/run_blender_indoor.sh` |
| Full Blender-indoor run      | `BLENDER=/path/to/blender INPUT_DIR=/path/to/blend_scenes bash scripts/run_blender_indoor.sh` |
| Summarize Blender-indoor run | `python3 scripts/summarize_blender_indoor_run.py --output-root outputs/blender_indoor` |
| Anonymity check              | `bash scripts/check_anonymity.sh`                            |

The smoke test is designed to run without private assets. The full Blender-indoor run requires local `.blend` scenes and a Blender executable.

## Repository Layout

```text
.
├── pipelines/               # full scene pipelines; Blender indoor is the primary path
├── scripts/                 # review and reproduction entry points
├── configs/                 # default and source-specific configs
├── core/                    # ERP projection, tangent extraction, depth, and warping modules
├── tools/                   # semantic and navigability helpers
├── utils/                   # IO and pose utilities
├── examples/                # tiny Blender-indoor-style metadata example
├── metadata_examples/       # JSON schemas for candidate/selection logs
├── data/                    # local data mount point, not tracked
├── third_party/             # optional external dependencies, not tracked
└── results/                 # generated result CSVs
```

## Environment

```bash
conda env create -f environment.yml
conda activate cmevs
```

If Conda is unavailable:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Minimal Smoke Test

```bash
bash scripts/run_tiny.sh
```

Expected outputs:

```text
outputs/tiny/metadata/candidates.jsonl
outputs/tiny/metadata/selected_viewpoints.json
outputs/tiny/metadata/per_step_log.jsonl
outputs/tiny/renders/
outputs/tiny/results/coverage_main.csv
outputs/tiny/results/oracle_validation.csv
outputs/tiny/results/audit_50_frames.csv
```

This test validates the repository wiring and metadata contracts. It is not intended to reproduce paper-scale numbers.

## Paper Experiments

The driver scripts for the §6 evaluation experiments (fixed-budget coverage, oracle-gain validation, λ sweep, cross-source robustness, downstream depth) are scheduled to be released alongside the camera-ready paper. The current release ships the algorithmic core (`scripts/build_candidates.py`, `scripts/select_views.py`, `scripts/render_selected.py`), the per-stage evaluation building blocks (`scripts/evaluate_coverage.py`, `scripts/evaluate_oracle_gap.py`, `scripts/audit_quality.py`), and the metadata-contract example through the smoke test. Reviewers can verify the algorithmic core end-to-end via the smoke test above.

## Primary Full Run: Blender Indoor

Put `.blend` scenes under `data/blender_indoor/`, or point `INPUT_DIR` to another directory. Nested layouts are supported; the first subdirectory under `INPUT_DIR` is used as the scene name.

Dry run:

```bash
DRY_RUN=1 \
BLENDER=/path/to/blender \
INPUT_DIR=data/blender_indoor \
OUTPUT_ROOT=outputs/blender_indoor \
bash scripts/run_blender_indoor.sh
```

Full run:

```bash
BLENDER=/path/to/blender \
INPUT_DIR=data/blender_indoor \
OUTPUT_ROOT=outputs/blender_indoor \
NUM_FRAMES=30 \
RESOLUTION=2048,1024 \
bash scripts/run_blender_indoor.sh
```

Equivalent direct CLI:

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

## Secondary Adapters

The repository also includes adapters for additional sources used in robustness analyses:

- `configs/blender_outdoor.yaml`: generic `.glb` / `.gltf` scenes.
- `configs/hm3d.yaml`: HM3D-style `.glb` / `.gltf` scenes.
- `configs/scannetpp.yaml`: ScanNet++-style `.ply` scenes.

These adapters are provided for completeness, but the Blender-indoor route is the recommended first reviewer path.

## Data and Checkpoints

This repository does not redistribute third-party scene assets, dataset files, or model checkpoints. Put local assets under `data/` or pass absolute paths via CLI. The `data/` directory is ignored by git.

Depth Pro is optional for ERPT-style depth fusion. If used, place it under:

```text
third_party/ml-depth-pro/
third_party/ml-depth-pro/checkpoints/depth_pro.pt
```

## Companion Dataset

The CM-EVS dataset (Blender indoor RGB-D panoramas, license-aware adapters for HM3D / ScanNet++ / OB3D / TartanGround, MLCommons Croissant metadata) is released on Hugging Face:

> **Dataset:** [`huggingface.co/datasets/anon-cmevs-2026/cmevs-erp-eval`](https://huggingface.co/datasets/anon-cmevs-2026/cmevs-erp-eval)
>
> **Hugging Face namespace (code and datasets):** <https://huggingface.co/anon-cmevs-2026>

The dataset card on that page includes the Datasheet, integrity-verification commands (`shasum -a 256 -c SHA256SUMS`), and the per-component license matrix. The Croissant manifest under `dataset_metadata/croissant.json` in this repository mirrors the dataset's metadata at release time.

## Final Submission Check

Before uploading the code URL or zip:

```bash
bash scripts/run_tiny.sh
rm -rf outputs
bash scripts/check_anonymity.sh
```

The code is released under the MIT License for review. Dataset assets remain governed by their original licenses.

## Links

- **Company website:** <https://www.aniverse.net.cn/>
- **GitHub organization:** <https://github.com/Strange-animalss>
- **Hugging Face:** <https://huggingface.co/anon-cmevs-2026> (code and datasets)
