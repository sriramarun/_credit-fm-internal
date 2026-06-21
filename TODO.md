# TODO / Backlog — Credit Foundation Model Framework

> Internal. Not pushed to git. Organized **by phase**, not by calendar day (dates drift;
> live status is here + in `PROGRESS.md`). `[x]` done · `[~]` partial · `[ ]` open.
>
> **Now:** Phase A — tokenizer (data split done). Target handoff window: early Sep 2026.

---

## Phase A — Setup + Tokenizer  ← CURRENT
- [x] Repo + `credit_fm` scaffold; Apache 2.0; CI.
- [x] H100 container env (venv, `pip install -e .[dev]`, 8 GPUs, W&B, HF).
- [x] Download dataset → `data/raw/Overall_2024_2025_all_months.parquet`; data scaffold + docs.
- [x] **Loan-stratified temporal split** (`prepare_data.py` + `splits.py` + tests) — derived origination.
- [x] Restore missing `src/credit_fm/data/` module (gitignore bug).
- [x] Classify the 71 ESMA fields → `tokenizer.yaml` (42 features, reproducible `find_redundant` + `--drop`, glossary 70/71, `find_redundant` CI test; PRs #9–#11).
- [x] `docs/decision_log.md` DL-001…DL-010 + `docs/tokenization.md` field-classification section.
- [x] Smoke test — splits validated via XGBoost (`notebooks/00_smoke_test_splits.ipynb`): config-1 ROC-AUC 0.93 on `all_cutoffs`, val≈test (honest baseline = Gate G1 in Phase B).
- [ ] `tokenizer/vocabulary.py` + `base.py` — build vocab **on `train.parquet` ONLY**.  ← NEXT
- [ ] `numeric_bucketer.py` (percentile + zero bucket); `categorical.py` ([UNK]); `temporal.py`.
- [ ] `KVTTokenizer` build_vocab/encode/decode; **roundtrip ≥99%**; token QA report.
- [ ] **M1: tokenizer done.**

## Phase B — Data layer + Baselines
- [x] `scripts/train_baseline.py` (4 configs, temporal split, performing-at-obs gate) + `reports/baseline_report.md`.
      **Gate G1 = ROC-AUC 0.734 / PR-AUC 0.046** (no-leakage + gate, on `all_cutoffs`).
- [ ] `label_generators.py` (default/prepay/cure) — formalize the baseline's forward-window label + edge-case tests.
- [ ] `evaluation/metrics.py` (ROC-AUC/PR-AUC/KS/Gini/Brier/lift) — match sklearn within 1e-6 (fold into baseline).
- [ ] Extract `loan_book.parquet` (`_segment`/`_latent_fragility`) → segment-conditional validation (XGBoost ceiling).
- [ ] `schema.py` validate; `dataset.py`; `collators.py` (packing + varlen); `datamodule.py` (wire dataset+tokenizer).

## Phase C — Model (three-branch encoder)
- [ ] `models/base.py` blocks (RoPE/attention/RMSNorm).
- [ ] `profile_encoder` (3L) · `event_encoder` (4–5L) · `history_encoder` (4–6L) + shape tests.
- [ ] `mlm_head` (3-vector concat) · `classification_head`.
- [ ] `credit_fm.py` composition; `training/masking.py` (15/10/10).
- [ ] Single-GPU validation (1k loans × 100 steps, non-NaN); `test_e2e` < 60s. **M2.**

## Phase D — Training + Pretraining
- [ ] `optimizers.py` + `callbacks.py` (W&B); `trainer.py` HF backend; NeMo adapter.
- [ ] Launch full pretraining on 500k Dutch mortgages (8× H100); monitor; iterate to convergence.
- [ ] Freeze candidate 30M checkpoint; training report. **M3: pretrained 30M (LFS).**

## Phase E — Inference + Evaluation + Dashboard
- [ ] `inference/extractor.py` + `pooling.py` ([USR]); `extract_embeddings.py`.
- [ ] `inference/lora.py` (rank 8); `evaluate_downstream.py` three-way (baseline/emb/combined/LoRA).
- [ ] `calibration.py` + `lift.py`; evaluation report.
- [ ] `ablation_profile_state.py` (expect +PR-AUC); FastAPI dashboard.
- [ ] Dutch mortgages model_card + data_card. **M4: Dutch ref complete.**

## Phase F — Invoice reference + Handoff
- [ ] Invoice `prepare_data` + `configs/invoice_financing/*`; document what changed vs Dutch.
- [ ] Train tokenizer + baseline + pretrain; extract + evaluate; cards. **M5.**
- [ ] Finalize `docs/`; coverage >80%; 5-min demo video; final technical report. **M6: handoff.**

---

## Open questions (track to closure)
1. Invoice financing data source + schema — needed before Phase F.
2. OSS release timing — full at end vs staged (checkpoint first)?
3. AML downstream — recommend documenting as out-of-scope (PRAGMA fails on it).
4. W&B hosted vs offline/self-hosted — resolve before Phase D (sovereign-cloud story).
5. Engagement continuation beyond month 3.
