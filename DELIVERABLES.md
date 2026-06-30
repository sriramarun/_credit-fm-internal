# Deliverables Manifest (INTERNAL)

> Internal tracking only — **do not commit to the public repo.** Maps the engagement's
> 17-component model-release taxonomy (Code / Data / Documentation) to repo locations + status.
> Paths are in `../credit-foundation-model/`.

**Status:** ✅ done · 🟡 partial / template · 🔴 scaffold or planned

## Code

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 1 | **Evaluation Code** | `scripts/train_baseline.py` (single-cutoff); `scripts/build_oot_baseline.py` (OOT); `scripts/evaluate_downstream.py` | 🟡 baselines ✅ (incl. real-world OOT); FM downstream eval stub |
| 2 | **Preprocessing Code** | `scripts/prepare_data.py`; `scripts/classify_schema.py`; `src/credit_fm/tokenizer/` (✅ M1); `src/credit_fm/data/` (dataset/collators/datamodule/encode) + `scripts/encode_dataset.py` (✅ M2 Brick 1) | ✅ **split + classify + tokenizer (M1) + data layer (M2 Brick 1) all done** — encode-once shards, `MLMCollator`, `CreditDataModule`, 44 tests |
| 3 | **Model Architecture** | `src/credit_fm/models/` (base RoPE/RMSNorm/SwiGLU; **hierarchical** profile/event/history encoders; 3-vector MLM head; `CreditFoundationModel`) | ✅ **M2 Brick 2 done (27 Jun)** — e2e overfits 3.92→0.10, 25.5M @ dim384; arch FROZEN, scaled at M3 |
| 4 | **Libraries & Tools** | `requirements.txt`, `pyproject.toml`, `Pipfile`, `scripts/setup_container.sh`, CI | ✅ done |
| 5 | **Training Code** | `scripts/pretrain.py`; `src/credit_fm/training/` (`masking.py` ✅ M2) | 🟡 masking ✅; trainer/optimizers/pretrain = M3 |
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

## Snapshot (as of M2 Brick 1 / data layer complete, 27 Jun)
✅ 4 done (Preprocessing #2, Libraries #4, Datasets #7, Evaluation Data #8) — #2 now covers split +
classify + tokenizer (M1) + the full data layer (M2 Brick 1, 44 tests) · 🟡 5 partial (baselines/
masking/metadata/configs/reports) · 🔴 ~6 scaffold/planned. **Next: M2 Brick 2 = hierarchical model
(`src/credit_fm/models/`), built + frozen at M2; M3 scales data only.**
