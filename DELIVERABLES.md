# Deliverables Manifest (INTERNAL)

> Internal tracking only — **do not commit to the public repo.** Maps the engagement's
> 17-component model-release taxonomy (Code / Data / Documentation) to repo locations + status.
> Paths are in `../credit-foundation-model/`.

**Status:** ✅ done · 🟡 partial / template · 🔴 scaffold or planned

## Code

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 1 | **Evaluation Code** | `scripts/train_baseline.py` (single-cutoff); `scripts/build_oot_baseline.py` (OOT); `scripts/evaluate_downstream.py` | 🟡 baselines ✅ (incl. real-world OOT); FM downstream eval stub |
| 2 | **Preprocessing Code** | `scripts/prepare_data.py`; `scripts/classify_schema.py`; `src/credit_fm/tokenizer/` (✅ M1); `src/credit_fm/data/` + `scripts/encode_dataset.py` (M2 Brick 1) | 🟡 split + classification + **tokenizer ✅ (M1, 440 tokens)**; data layer = M2 Brick 1 |
| 3 | **Model Architecture** | `src/credit_fm/models/` (**hierarchical** profile/event/history encoders, MLM head, `credit_fm.py`) | 🔴→🟡 **M2 in progress (hier., started 27 Jun); frozen at M2, scaled at M3** |
| 4 | **Libraries & Tools** | `requirements.txt`, `pyproject.toml`, `Pipfile`, `scripts/setup_container.sh`, CI | ✅ done |
| 5 | **Training Code** | `scripts/pretrain.py`; `src/credit_fm/training/` | 🔴 scaffold |
| 6 | **Inference Code** | `scripts/extract_embeddings.py`, `scripts/score_portfolio.py`; `src/credit_fm/inference/` | 🔴 scaffold |

## Data

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 7 | **Datasets** | **Fannie Mae SF Loan Performance (PRIMARY, real-world, 26y, GCS `gs://sriram-credit-fm-data`)** via `fannie-mae-etl` repo; `Algoritmica/green-lion-2024-2025` (HF, validation) | ✅ available + ingested |
| 8 | **Evaluation Data** | `data/processed/{val,test}.parquet` + `splits.csv` + `splits.meta.json` | ✅ done (temporal split, DL-007) |
| 9 | **Sample Model Outputs** | embedding parquet (Phase E) + portfolio scores | 🔴 planned |
| 10 | **Model Weights & Parameters** | `models/dutch_mortgages_30m/` (Git LFS) | 🔴 planned (post-pretraining) |
| 11 | **Model Metadata** | `configs/fannie_mae/tokenizer.json` ✅ (M1); `models/*/config.yaml`; W&B run metadata | 🟡 tokenizer.json ✅; model config/W&B = M2/M3 |
| 12 | **Configuration File** | `configs/fannie_mae/{baseline,raw_schema}.yaml`; `configs/dutch_mortgages/*.yaml`; `configs/invoice_financing/*` | 🟡 Fannie baseline configs ✅; model/training configs scaffold |

## Documentation

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 13 | **Data Card** | `reference_implementations/dutch_mortgages/data_card.md` | 🟡 template |
| 14 | **Research Paper** | technical writeup (PRAGMA-synthesis), Phase F | 🔴 planned |
| 15 | **Evaluation Results** | `reports/fannie_oot_crisis.md` (**ROC 0.757 / PR 0.024**), `reports/fannie_oot_recent.md` (ROC ~0.78), `reports/baseline_report.md` (Dutch 0.73); `reports/downstream_eval.md` (FM, planned) | 🟡 real-world baselines ✅ |
| 16 | **Model Card** | `docs/model_cards/dutch_mortgages.md`; `reference_implementations/dutch_mortgages/model_card.md` | 🟡 template |
| 17 | **Technical Report** | `reports/final_handoff/` (Phase F) | 🔴 planned |

## Per-phase production
- **A** (tokenizer): #2 preprocessing ✅, #4 libs ✅, #12 config ✅. **DONE (M1, 26 Jun).**
- **B** (data + baselines): #8 eval data ✅, #1/#15 baseline ✅.
- **C** (model = **M2**): #2 data layer (Brick 1) + #3 **hierarchical** architecture (Brick 2). *Architecture frozen at M2.*
- **D** (pretraining = **M3, scale only**): #5 training, #10 weights, #11 metadata. *Same arch as M2; tokenizer v2 re-fit on full corpus.*
- **E** (inference + eval): #6 inference, #9 sample outputs, #1/#15 downstream eval, #16 model card.
- **F** (handoff): #13 data card, #14 research paper, #17 technical report.

## Snapshot (as of M1 tokenizer complete, 27 Jun)
✅ 3 done (Libraries, Datasets, Evaluation Data) + tokenizer/M1 landed within #2/#11/#12 ·
🟡 6 partial · 🔴 ~7 scaffold/planned. **Next: M2 = data layer + hierarchical model (architecture frozen at M2; M3 scales data only).**
