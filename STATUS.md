# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 27 Jun 2026)*

**📌 27 Jun — M2 Brick 1 (DATA LAYER) DONE + merged.** The full pretraining data pipeline is built,
tested, and on `main` (PRs #34–#39): `training/masking.py` (3-source MLM 15/10/10), `encode_with_meta`
+ `scripts/encode_dataset.py` (encode-once token-id **shards** + manifest — run on real Fannie),
`CreditSequenceDataset`, `MLMCollator` (flat `(B,L)` pad+mask), `CreditDataModule` (train/val/test
loaders). **44 tests green, ruff clean.** Docs refreshed (detailed `architecture.md`, `RUNBOOK.md`
run steps, DL-011…014). **Next: M2 Brick 2** — the hierarchical three-branch model (arch frozen at M2).

**📌 26 Jun — M1 tokenizer DONE.** KVT tokenizer fit on the real Fannie train split (25.6M rows) →
**440-token** vocab; re-fit with reviewer #1/#2 incorporated (threshold-anchored + per-field bins;
`cal=<YYYYQ#>` macro-regime token). 100% lossless roundtrip, 0% OOV; merged to `main` (PR #31 code,
PR #32 artifacts), tests 14/14. ✅ Repo docs updated for the new tokens (PR #33, #39).

**📌 25 Jun — real-world Fannie baselines done; the bar for the FM is set.** The full 26-yr Fannie
book is ingested to GCS, and two honest **out-of-time (OOT)** baselines are built. Baseline phase
is essentially complete; the model itself (tokenizer → encoders → pretrain) is the critical path now.

## At a glance

| | |
|---|---|
| **Overall** | 🟢 On track (ahead of plan; tokenizer + data layer done, hierarchical model next) |
| **Phase** | A + B done; **Phase C / M2 in progress** — data layer ✅, model (Brick 2) next (week ~2 of ~12) |
| **Next milestone** | **M2** model forward + toy train — target **29 Jul 2026** (internal aim ~18 Jul; Brick 1 done 27 Jun) |
| **Key metric (the bar)** | **Fannie OOT crisis: ROC 0.757 / PR-AUC 0.024** (real-world, leakage-free). Recent (2023–24): ROC ~0.78. |
| **Biggest unknown** | Will the FM train *and* beat ~0.76 OOT? (data layer ready; model code next) |

## Milestone progress

```
M1 tokenizer      ▓▓▓▓▓▓▓▓▓▓  DONE — fit on real Fannie train (25.6M rows) → 440-token vocab; anchored/per-field bins + cal= macro token; 100% roundtrip, 0% OOV; merged (PR #31/#32), tests 14/14  (done 26 Jun, ahead of 1 Jul)
G1 baseline gate  ▓▓▓▓▓▓▓▓▓▓  DONE — real-world OOT bars established (crisis + recent)
M2 model          ▓▓▓░░░░░░░  ~30%   Brick 1 DATA LAYER ✅ MERGED (masking/encode-once shards/dataset/MLMCollator flat (B,L)/datamodule; PRs #34–39; 44 tests). Next Brick 2 = hierarchical model (base→event→profile→history→heads). arch FROZEN at M2   aim ~18 Jul / target 29 Jul
M3 pretrained     ░░░░░░░░░░  not started   target 12 Aug
M4 Fannie ref     ░░░░░░░░░░  not started   target 26 Aug
M5/M6 handoff     ░░░░░░░░░░  not started   target  9 Sep
```

## This week (cadence: Last / This / Next)

**Done (weeks 1–2)**
- 8× H100 container + private repo + CI; pluggable storage (`credit_fm.utils.storage`: local/gs://s3://).
- **Fannie data pipeline:** raw zip → parquet → reporting-partitioned GCS (`fannie-mae-etl` repo,
  rewritten ETL: 104 files in **28 min** vs ~10h). `ingest_fannie_mae.py` + `prepare_data.py` (pluggable).
- **Single-cutoff baseline machinery** (`train_baseline.py`, config-driven) — Dutch Gate-G1 = ROC 0.73.
- **OOT (calendar-split) baseline** (`build_oot_baseline.py`) — the real deliverable:
  - **Crisis (train 2000–06 → test 2008–10): ROC 0.757 / PR-AUC 0.024** ✅ committed.
  - **Recent (→ 2023–25): ROC 0.784** — censored 2025 cohort; **2023–24 re-run pending**.
  - Guards: loan-disjoint + embargo; GPU xgboost; 20% loan sample; 57 no-leakage features.
- **Dutch `_segment` ceiling proof** retained as the FM thesis (synthetic, controlled).

**Done (week ~2)**
- **M1 tokenizer COMPLETE** — `KVTTokenizer` fit on real Fannie train → 440-token vocab; reviewer #1
  (anchored + per-field bins) and #2 (`cal=<YYYYQ#>` macro token) shipped; `tokenizer.json` + report.
- **M2 Brick 1 — DATA LAYER COMPLETE (27 Jun, PRs #34–39):** 3-source MLM masking (15/10/10);
  `encode_with_meta` + `encode_dataset.py` (encode-once shards + manifest, run on real Fannie);
  `CreditSequenceDataset`; `MLMCollator` (flat `(B,L)` pad+mask); `CreditDataModule`. **44 tests green.**
  Frozen contract: `input_ids`/`event_index`/`field_type`/`branch` per token.
- **Docs refreshed** — detailed `docs/architecture.md`, `RUNBOOK.md` run steps, DL-011…014 (PR #33, #39).

**Now / next**
- Start **M2 Brick 2 — hierarchical model**: `models/base.py` (RoPE/RMSNorm/SwiGLU) → `event_encoder`
  (intra-event, pools via `event_index`) → `profile_encoder` → `history_encoder` (`[USR]` pooling) →
  heads → `credit_fm.py`; `test_e2e` fwd+bwd <60s on one toy batch. Context window **1024**.
- Then the toy train loop (1k loans × 100 steps, loss ↓) → **M2 gate, architecture FROZEN**.
- Finish the **2023–24 recent OOT re-run** (drop censored 2025) → clean recent number.

## Key result — the story now

> On **real-world** Fannie data, an honest out-of-time tabular model (no leakage, loan-disjoint,
> trained only on the past) ranks new defaults at **ROC ≈ 0.76–0.78** — credible, in line with
> industry mortgage-PD models, and a *fair* bar. PR-AUC is low (rare events: ~3.3× over base rate
> in both regimes) precisely because the high-octane delinquency signal is excluded. The crisis
> cut (2008–10) is the hardest test. The foundation model reads each loan's **behavioural
> sequence** to push ROC toward the mid-0.80s — **that gap is the headroom, now measured on real data.**
> (Dutch synthetic `_segment` ceiling — +95% PR-AUC oracle lift — remains the controlled proof of concept.)

## Blockers & asks
- 🟡 **2023–24 recent OOT re-run** not yet committed (one nohup run). Optional: add auto-censoring guard.
- 🟡 **W&B hosting decision** (hosted vs offline/self-hosted) before pretraining. *(DL-009)*
- 🟡 **Invoice-financing dataset** — needed by ~week 9. *(Algoritmica)*

## Health notes
- **Schedule:** **ahead** of plan — M1 (tokenizer) + M2 Brick 1 (data layer) both done by 27 Jun vs
  M1 target 1 Jul / M2 target 29 Jul. The early slack is being banked as M3 (pretraining) buffer.
- **Risk concentration:** ~80% of technical uncertainty is still in the unbuilt model + pretraining
  (Phases C–D). Data plumbing is now solid; the open question — does the hierarchical model train and
  beat ~0.76 — is exactly what M2 Brick 2 + the toy loop will answer next.
- **De-risking posture:** architecture is debugged at toy scale and **frozen at M2**; M3 only scales
  data/compute — so we never debug architecture and distributed convergence at the same time.
- **Process:** git/file-handoff friction reduced via heredoc workflow + `git ls-files` verify habit;
  M2 Brick 1 shipped as 4 stacked PRs cleanly.
