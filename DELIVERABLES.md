# Deliverables Manifest (INTERNAL)

> Internal tracking only — **do not commit to the public repo.** Maps the engagement's
> 17-component model-release taxonomy (Code / Data / Documentation) to repo locations + status.
> Paths are in `../credit-foundation-model/`.

**Status:** ✅ done · 🟡 partial / template · 🔴 scaffold or planned

## Code

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 1 | **Evaluation Code** | `src/credit_fm/evaluation/`; `scripts/train_baseline.py`; `scripts/evaluate_downstream.py` | 🟡 baseline done; metrics/benchmarks stub |
| 2 | **Preprocessing Code** | `scripts/prepare_data.py`; `scripts/classify_schema.py`; `src/credit_fm/data/`; `src/credit_fm/tokenizer/` | 🟡 split + classification done; tokenizer pending |
| 3 | **Model Architecture** | `src/credit_fm/models/` (profile/event/history encoders, MLM head, `credit_fm.py`) | 🔴 scaffold |
| 4 | **Libraries & Tools** | `requirements.txt`, `pyproject.toml`, `Pipfile`, `scripts/setup_container.sh`, CI | ✅ done |
| 5 | **Training Code** | `scripts/pretrain.py`; `src/credit_fm/training/` | 🔴 scaffold |
| 6 | **Inference Code** | `scripts/extract_embeddings.py`, `scripts/score_portfolio.py`; `src/credit_fm/inference/` | 🔴 scaffold |

## Data

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 7 | **Datasets** | `Algoritmica/green-lion-2024-2025` (HF); `data/raw/` (gitignored); `data/README.md` | ✅ available |
| 8 | **Evaluation Data** | `data/processed/{val,test}.parquet` + `splits.csv` + `splits.meta.json` | ✅ done (temporal split, DL-007) |
| 9 | **Sample Model Outputs** | embedding parquet (Phase E) + portfolio scores | 🔴 planned |
| 10 | **Model Weights & Parameters** | `models/dutch_mortgages_30m/` (Git LFS) | 🔴 planned (post-pretraining) |
| 11 | **Model Metadata** | `models/*/config.yaml`, `tokenizer.json`; W&B run metadata | 🟡 placeholders |
| 12 | **Configuration File** | `configs/dutch_mortgages/*.yaml`; `configs/invoice_financing/*` | 🟡 `tokenizer.yaml` done; rest scaffold |

## Documentation

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 13 | **Data Card** | `reference_implementations/dutch_mortgages/data_card.md` | 🟡 template |
| 14 | **Research Paper** | technical writeup (PRAGMA-synthesis), Phase F | 🔴 planned |
| 15 | **Evaluation Results** | `reports/baseline_report.md` (Gate G1: ROC 0.73 / PR-AUC 0.046); `reports/downstream_eval.md` | 🟡 baseline done |
| 16 | **Model Card** | `docs/model_cards/dutch_mortgages.md`; `reference_implementations/dutch_mortgages/model_card.md` | 🟡 template |
| 17 | **Technical Report** | `reports/final_handoff/` (Phase F) | 🔴 planned |

## Per-phase production
- **A** (tokenizer): #2 preprocessing, #4 libs, #12 config.
- **B** (data + baselines): #8 eval data ✅, #1/#15 baseline ✅.
- **C** (model): #3 architecture.
- **D** (pretraining): #5 training, #10 weights, #11 metadata.
- **E** (inference + eval): #6 inference, #9 sample outputs, #1/#15 downstream eval, #16 model card.
- **F** (handoff): #13 data card, #14 research paper, #17 technical report.

## Snapshot (as of baseline complete)
✅ 3 done (Libraries, Datasets, Evaluation Data) · 🟡 6 partial · 🔴 8 scaffold/planned.
