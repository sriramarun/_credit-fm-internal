# Deliverables Manifest (INTERNAL)

> Internal tracking only — **do not commit to the public repo.** Maps the engagement's
> 17-component model-release taxonomy (Code / Data / Documentation) to repo locations + status.
> Paths are in `../credit-foundation-model/`.

**Status:** ✅ done · 🟡 partial / template · 🔴 scaffold or planned

## Code

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 1 | **Evaluation Code** | `scripts/train_baseline.py`; `scripts/build_oot_baseline.py`; `scripts/evaluate_downstream.py` (features/emb/combined XGB + linear probe); `scripts/finetune.py` (frozen/LoRA/full) | ✅ **done (2 Jul)** — all config-driven |
| 2 | **Preprocessing Code** | `scripts/prepare_data.py`; `scripts/classify_schema.py`; `src/credit_fm/tokenizer/` (✅ M1); `src/credit_fm/data/` (dataset/collators/datamodule/encode) + `scripts/encode_dataset.py` (✅ M2 Brick 1) | ✅ **split + classify + tokenizer (M1) + data layer (M2 Brick 1) all done** — encode-once shards, `MLMCollator`, `CreditDataModule`, 44 tests |
| 3 | **Model Architecture** | `src/credit_fm/models/` (base RoPE/RMSNorm/SwiGLU; **hierarchical** profile/event/history encoders; 3-vector MLM head; `CreditFoundationModel`) | ✅ **M2 Brick 2 done (27 Jun)** — e2e overfits 3.92→0.10, 25.5M @ dim384; arch FROZEN, scaled at M3 |
| 4 | **Libraries & Tools** | `requirements.txt`, `pyproject.toml`, `Pipfile`, `scripts/setup_container.sh`, CI | ✅ done |
| 5 | **Training Code** | `scripts/pretrain.py` (config recipe); `src/credit_fm/training/` (masking, optimizers, `train_mlm` best-val); `scripts/finetune.py` (PRAGMA adaptation ladder) | ✅ **done (2 Jul)** — M3 full pretrain ran (453M tokens); DDP deliberately last |
| 6 | **Inference Code** | `scripts/extract_embeddings.py` ([USR] pooling + leakage gates, config-driven) ✅; `scripts/score_portfolio.py` 🔴 stub | 🟡 extract ✅; batch scoring demo planned |

## Data

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 7 | **Datasets** | **Fannie Mae SF Loan Performance (PRIMARY, real-world, 26y, GCS `gs://sriram-credit-fm-data`)** via `fannie-mae-etl` repo; `Algoritmica/green-lion-2024-2025` (HF, validation) | ✅ available + ingested |
| 8 | **Evaluation Data** | `data/processed/{val,test}.parquet` + `splits.csv` + `splits.meta.json` | ✅ done (temporal split, DL-007) |
| 9 | **Sample Model Outputs** | embedding parquet exists on GCS (`m3_dec2016_train.parquet`) but not packaged as a repo sample; portfolio scores | 🔴 packaging planned |
| 10 | **Model Weights & Parameters** | `runs/m3_full.pt` on GCS (25.5M, trained 453M tokens) — **not yet published** (LFS/HF Hub + license) | 🟡 checkpoint exists; publish planned |
| 11 | **Model Metadata** | `configs/fannie_mae/tokenizer.json` ✅ (M1); `models/*/config.yaml`; W&B run metadata | 🟡 tokenizer.json ✅; model config/W&B = M2/M3 |
| 12 | **Configuration File** | `configs/fannie_mae/` — `common.yaml` + 11 stage recipes (ingest/prepare/classify/tokenizer_fit/baseline_run/oot_baseline/encode/pretrain/extract/evaluate/finetune) + generated `tokenizer.yaml`/`baseline.yaml`/`raw_schema.yaml` | ✅ **done (2 Jul, PRs #53/#54)** — config engine w/ include, ${a.b}, dotted overrides, lineage |

## Documentation

| # | Deliverable | Location | Status |
|---|-------------|----------|--------|
| 13 | **Data Card** | `reference_implementations/dutch_mortgages/data_card.md` | 🟡 template |
| 14 | **Research Paper** | technical writeup (PRAGMA-synthesis), Phase F | 🔴 planned |
| 15 | **Evaluation Results** | `reports/fannie_oot_crisis.md` (**0.757 bar**), `reports/fannie_oot_recent.md` (~0.78), `reports/fm_downstream_dec2016.md` (frozen probe: 0.687 vs 0.746), `reports/ft_{frozen,full,lora}.md` (**LoRA 0.7526 > 0.746**) | 🟡 recent-window done; crisis-OOT FM verdict open |
| 16 | **Model Card** | `docs/model_cards/dutch_mortgages.md`; `reference_implementations/dutch_mortgages/model_card.md` | 🟡 template |
| 17 | **Technical Report** | `reports/final_handoff/` (Phase F) | 🔴 planned |

## Per-phase production
- **A** (tokenizer): #2 preprocessing ✅, #4 libs ✅, #12 config ✅. **DONE (M1, 26 Jun).**
- **B** (data + baselines): #8 eval data ✅, #1/#15 baseline ✅.
- **C** (model = **M2**): #2 data layer (Brick 1) + #3 **hierarchical** architecture (Brick 2). *Architecture frozen at M2.*
- **D** (pretraining = **M3, scale only**): #5 training, #10 weights, #11 metadata. *Same arch as M2; tokenizer v2 re-fit on full corpus.*
- **E** (inference + eval): #6 inference, #9 sample outputs, #1/#15 downstream eval, #16 model card.
- **F** (handoff): #13 data card, #14 research paper, #17 technical report.

## Snapshot (as of fine-tune verdict + config-driven framework, 2 Jul)
✅ 8 done (#1 evaluation, #2 preprocessing, #3 architecture, #4 libraries, #5 training, #7 datasets,
#8 eval data, #12 configuration) · 🟡 5 partial (#6 inference, #10 weights-unpublished, #11 metadata,
#13/#16 cards-as-templates, #15 results) · 🔴 4 planned (#9 sample outputs, #14 paper, #17 tech
report, batch scoring). **Release-critical gaps, in order: model+data cards → weights publish →
notebooks → sample outputs → crisis-OOT verdict → technical report/paper. DDP deliberately last.**
