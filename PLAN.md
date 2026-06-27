# Project Plan — Credit Foundation Model Framework

> Internal · not pushed to git. **The stable map** (changes rarely). For "where are we right
> now" see [`STATUS.md`](STATUS.md). Tasks: `[x]` done · `[~]` partial · `[ ]` open.

**Objective.** Open-source (Apache 2.0) framework for training credit foundation models +
reference implementations. finevals.ai × Sriram Krishnan, NVIDIA-sponsored (8× H100).
Encoder-only MLM, three-branch, key-value-time tokenization.

**Primary corpus (decided 22 Jun 2026): Fannie Mae Single-Family Loan Performance** — real-world
US fixed-rate mortgages, ~25 years, ~100 quarterly parquet snapshots from GCS. Dutch synthetic
panel → **validation/ablation** (keeps the `_segment` ceiling proof); invoice = planned. Strategy:
dev-sample-first → **now at full 26-yr scale on GCS; OOT baselines done (25 Jun)**. M3/M4 on Fannie.
**Two GCS layouts:** `parquet/<acqQ>` (cohort, full loan lives → OOT baseline + FM sequences) ·
`fannie_by_reporting/` (reporting-partitioned → single-cutoff snapshot baseline).

**Timeline.** Kickoff **Thu 18 Jun 2026**, ~12 weeks (6 phases × 2 weeks) → handoff **~9 Sep 2026**.

## Milestones (target dates)

| Milestone | Phase | Target date | Status |
|-----------|-------|-------------|--------|
| **M1** — tokenizer complete | A | **Wed 1 Jul 2026** | 🟢 **DONE early (26 Jun)** — 440-token vocab on real Fannie train; anchored bins + `cal=` macro token; merged PR #31/#32 |
| **G1** — baseline gate (trustworthy labels + strong baseline) | B | **Tue 15 Jul 2026** | 🟢 **DONE early** — real-world Fannie **OOT** bars (crisis ROC 0.757 / PR 0.024; recent ~0.78) |
| **M2** — **hierarchical** model forward + toy train works (architecture frozen) | C | **Tue 29 Jul 2026** (internal aim ~18 Jul) | ⬜ |
| **M3** — pretrained 30M checkpoint (M2 arch scaled — data/compute only) | D | **Tue 12 Aug 2026** | ⬜ |
| **M4** — Fannie mortgages reference complete | E | **Tue 26 Aug 2026** | ⬜ |
| **M5 / M6** — invoice reference + final handoff | F | **Tue 9 Sep 2026** | ⬜ |

## Task tracker (by phase)

### Phase A — Setup + Tokenizer  (18 Jun → 1 Jul)
- [x] Repo + `credit_fm` scaffold; Apache 2.0; CI.
- [x] H100 container env (venv, install, 8 GPUs, W&B, HF).
- [x] Loan-stratified **temporal** split (`prepare_data.py` + `splits.py` + tests).
- [x] Restore missing `src/credit_fm/data/` module (gitignore bug).
- [x] Reproducible 71-field classification → `tokenizer.yaml` (42 features; `find_redundant` + CI test).
- [x] `docs/decision_log.md` DL-001…010; `docs/tokenization.md`; `CLAUDE.md`.
- [x] Smoke test validates splits (`notebooks/00_smoke_test_splits.ipynb`).
- [x] `tokenizer/vocabulary.py` + `numeric_bucketer.py` · `categorical.py` ([UNK]) — vocab fit on `train` only.
- [x] `KVTTokenizer` encode/decode (branch routing, [USR]/[EVT] blocks, save/load); `train_tokenizer.py` + `tokenizer.json` + token QA report (100% roundtrip). → **M1 essentially done (26 Jun)**.
- [x] **Reviewer #1/#2 incorporated + re-fit** — threshold-aware + per-field bins (`anchors`, `bins`), calendar `cal=<YYYYQ#>` token (macro regime). ✅ `tokenizer.json` **re-fit on real Fannie train (25.6M rows) → 440 tokens**, merged (PR #31 code / #32 artifacts). Real macro (HPI/rate/unemp) = future event fields. → **M1 DONE (26 Jun)**.
- [ ] **Doc debt (M1):** update `docs/tokenization.md` (fused `field=value`, `t=`+`cal=`, anchored bins — current text is stale log-seconds/BPE) and `docs/decision_log.md` (DL-011 calendar/macro token, DL-012 anchored/per-field bins).

### Phase B — Data layer + Baselines  (2 Jul → 15 Jul)  — **baselines DONE (ahead of plan)**
- [x] **Fannie Mae (PRIMARY) data pipeline** — `fannie-mae-etl` repo (raw zip → parquet → reporting-partitioned GCS, 104 files in 28 min); `ingest_fannie_mae.py` (hive read → derived `default_event`/`is_performing`/ISO dates); `prepare_data.py` + pluggable `storage` (local/gs://s3://); `configs/fannie_mae/{baseline,raw_schema}.yaml`; `docs/data/fannie_mae.md`.
- [x] **OOT (calendar-split) baseline** (`scripts/build_oot_baseline.py`) — the real bar: per-loan Dec snapshots, 12-mo forward default, loan-disjoint + embargo guards, GPU xgboost, 20% sample, 57 no-leakage features. **Crisis 2000–06→2008–10 = ROC 0.757 / PR 0.024**; **recent →2023–25 = ROC 0.784** (2023–24 re-run pending to drop censored 2025).
- [x] `scripts/train_baseline.py` (single-cutoff, 4 configs, performing-gate) + `reports/baseline_report.md` → **Gate G1 (Dutch synthetic) = ROC 0.73 / PR-AUC 0.046** (methodology proof).
- [x] **Ceiling validation wired into `train_baseline.py` + report** — segment-conditional default (34× spread), oracle-segment lift (PR-AUC +95%, 0.046→0.090), segment unrecoverable from observables (65% ≈ 63% majority). `loan_book.parquet` = `out_500k_v2_1`.
- [x] **`train_baseline.py` is now config-driven** (`configs/dutch_mortgages/baseline.yaml`) → schema-agnostic; new asset = new config, no code change. Split + classify already generic. `RUNBOOK.md` added.
- [ ] (now in service of the FM, not the baseline) `evaluation/metrics.py` (ROC/PR/KS/Gini/Brier/lift) — sklearn-parity; `label_generators.py` formalize forward-window label.
- [→] `data/` sequence loader (`dataset.py`/`collators.py`/`datamodule.py`) — **moved to Phase C / M2 Brick 1** (it's the model's data layer; see below).
- [ ] **Cleanups:** commit 2023–24 recent OOT re-run; optional auto-censoring guard; drop `reports/_oot_smoke.md`.

### Phase C — Model (HIERARCHICAL three-branch encoder) — M2  (start 27 Jun · internal aim ~18 Jul · target 29 Jul)
**Decision (27 Jun): hierarchical from the start.** Build + debug the *full* hierarchical architecture
at SMALL scale here, then **freeze it**. M3 changes **only data volume / compute** — never architecture.
(→ promote to repo **DL-013** when the model code lands.) Goal of M2 = *evidence the architecture learns*,
not a good model.

**Brick 1 — Data layer** (`src/credit_fm/data/`; encode-ONCE so the dataloader never re-tokenizes)
- [ ] `scripts/encode_dataset.py` — tokenizer.json + processed parquet → token-id **shards** on GCS (`output/encoded/fannie_mae/...`).
- [ ] `dataset.py` (mmap shard reader → ids + branch/segment ids + event boundaries) · `collators.py` (pad + attn mask + `labels=-100` on unmasked) · `datamodule.py` (loaders; length-bucketed sampler deferred to M3).
- [ ] `training/masking.py` — **15% token / 10% whole-event / 10% whole-field-type** (structured, TabBERT/PRAGMA); never mask specials.
- [ ] tests `test_dataset`/`test_collators`/`test_masking`. **Gate:** batch → `(B,L)` ids/mask/labels/branch_ids, round-trips tokenizer, mask fractions correct.

**Brick 2 — Hierarchical model** (`src/credit_fm/models/`)
- [ ] `base.py` — transformer block (attention + **RoPE**, **RMSNorm**, SwiGLU).
- [ ] `profile_encoder` (~3L) → profile vector · `event_encoder` (~4–5L, **intra-event**) → per-event vector · `history_encoder` (~4–6L, **across event vectors** + profile; `[USR]` pools the loan embedding) + shape tests. *(Hierarchy = history length is #events not #tokens → cheaper, more faithful than a flat model.)*
- [ ] `heads.py` — `mlm_head` (→ 440 vocab) · `classification_head` (`[USR]` → default logit).
- [ ] `credit_fm.py` composition; **~30M params**. `test_e2e` (fwd+bwd, finite, loss decreasing, **<60s**).

**Brick 3 — Toy train + FREEZE**
- [ ] Single-GPU **1k loans × 100 steps**, loss trending down; `ruff`+`pytest` clean. → **M2 — architecture working + FROZEN.**

### Phase D — Pretraining at scale — M3 (architecture FROZEN; turn up data only)  (30 Jul → 12 Aug)  ← highest risk
**M3 = the *exact* M2 hierarchical architecture, scaled: full corpus + 8×H100 + ~600M tokens.
No architecture changes — only data/compute + training infra + the at-scale tokenizer freeze.**
- [ ] `optimizers.py` (AdamW, warmup-cosine, **bf16**, grad-clip) + `callbacks.py` (W&B, GCS/LFS checkpointing); `trainer.py` (HF Accelerate, **8×H100 DDP**) + NeMo adapter.
- [ ] **Tokenizer v2 re-fit** on a multi-vintage sample (incl. 2006–2010 crisis values) **before** freezing vocab for pretrain; **version it** (reviewer #8). Then `encode_dataset.py` over the **full corpus** → shards (~600M tokens / ~1.2M loans, Chinchilla-honest for 30M).
- [ ] Full pretraining on **Fannie sequences** (8× H100); monitor val loss + masked-token accuracy; iterate to convergence.
- [ ] **Reviewer #4 — length-bucketed batching** in `collators.py` (group similar-length loans → far less padding; M3 throughput; bounded by `max_events=60`).
- [ ] **Macro / context features (deferred from 27 Jun discussion)** — join public point-in-time series at **state/MSA** granularity as *loan-relative derived* event fields: FHFA HPI → **mark-to-market LTV** (negative equity, the dominant default driver); PMMS/FRED rate → **refi incentive**; BLS unemployment → ability-to-pay. **Vintage/as-of join only** (no revised series, respect publish lag — leakage discipline). **Give the OOT baseline the identical features** (expect the 0.757 crisis bar to *rise*; the FM's edge must then come from sequence dynamics, not macro levels). Reusable "context branch" across products. *Decide at Phase D start: include in the first pretrain, or add post-checkpoint as an ablation.*
- [ ] Freeze candidate 30M checkpoint; training report. → **M3 (LFS)**

### Phase E — Inference + Evaluation + Dashboard  (13 Aug → 26 Aug)
- [ ] `inference/extractor.py` + `pooling.py` ([USR]); `extract_embeddings.py`.
- [ ] `inference/lora.py`; `evaluate_downstream.py` — **FM embeddings through the SAME OOT split/label/metric as `build_oot_baseline.py`** (the FM-vs-0.757 test), three-way (baseline / emb / combined / LoRA).
- [ ] **Reviewer #3 — class imbalance** on the downstream head: embedding-probe inherits the baseline's handling (PR-AUC, `--neg-per-pos`); if **fine-tuning**, add focal loss / class weights. **Fairness:** give the OOT baseline the same calendar/macro features the FM gets, or the win isn't apples-to-apples.
- [ ] `calibration.py` + `lift.py`; evaluation report; `ablation_profile_state.py`.
- [ ] FastAPI dashboard; Dutch mortgages model_card + data_card. → **M4**

### Phase F — Invoice reference + Handoff  (27 Aug → 9 Sep)
- [ ] Invoice `prepare_data` + `configs/invoice_financing/*`; document config deltas vs Dutch.
- [ ] Train tokenizer + baseline + pretrain; extract + evaluate; cards. → **M5**
- [ ] Finalize docs; coverage >80%; 5-min demo; final technical report. → **M6 handoff**

## Critical path
~~B/G1 (baseline gate)~~ **DONE** → **A (tokenizer) = current front** → C (model) →
**D (pretraining run = longest pole, protect it)** → E depends on a converged checkpoint;
E's downstream eval reuses the OOT baseline harness. F depends on external invoice data.

## Risks
| Risk | Mitigation |
|------|------------|
| Will the model train/converge at all? (biggest unknown) | Race to a toy run early (Phase C); checkpoint often |
| Memorization (30M on ~600M synthetic tokens) | Hold 30M; 50M only with extra public data; watch val loss |
| Inflated baseline → misleading lift | Honest gated/no-leakage baseline (done); segment-ceiling context |
| Data leakage (future-state fields) | loan-level temporal split; vocab on train; gate state features |
| Invoice data slips | Escalate to Algoritmica now; Dutch ref is the primary deliverable |
| H100 access/storage | Checkpoint frequently; size run to available GPUs |

## Open questions
1. **Invoice financing data** — source + schema, needed before Phase F. *(Algoritmica)*
2. **Latent-segment extract** — `loan_book.parquet` (located locally; get it onto the container). *(data team)*
3. **W&B hosted vs offline/self-hosted** — decide before Phase D (DL-009; sovereign-cloud).
4. **OSS release timing** — full at end vs staged (checkpoint first)?
5. **Engagement continuation** beyond month 3.

## Existing assets (don't rebuild)
`Algoritmica/green-lion-2024-2025` (HF) · deeploans synthetic-data-designer · 53-test SQL
validation suite · NVIDIA TFM blueprint (adapt, don't fork) · FastAPI dashboard skeleton.
