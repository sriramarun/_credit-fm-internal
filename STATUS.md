# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 25 Jun 2026)*

**📌 25 Jun — real-world Fannie baselines done; the bar for the FM is set.** The full 26-yr Fannie
book is ingested to GCS, and two honest **out-of-time (OOT)** baselines are built. Baseline phase
is essentially complete; the model itself (tokenizer → encoders → pretrain) is the critical path now.

## At a glance

| | |
|---|---|
| **Overall** | 🟢 On track (baselines ahead of plan; model not started) |
| **Phase** | A→B done-ish — Data + Baselines; next is the model (week ~2 of ~12) |
| **Next milestone** | **M1** tokenizer — target **1 Jul 2026** (schema done; code not started) |
| **Key metric (the bar)** | **Fannie OOT crisis: ROC 0.757 / PR-AUC 0.024** (real-world, leakage-free). Recent (2023–24): ROC ~0.78. |
| **Biggest unknown** | Will the FM train *and* beat ~0.76 OOT? (no model code exists yet) |

## Milestone progress

```
M1 tokenizer      ▓▓░░░░░░░░  ~20%  schema/classification done, tokenizer CODE not started  target 1 Jul
G1 baseline gate  ▓▓▓▓▓▓▓▓▓▓  DONE — real-world OOT bars established (crisis + recent)
M2 model          ░░░░░░░░░░  not started   target 29 Jul
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

**Now / next**
- Finish the **2023–24 recent OOT re-run** (drop censored 2025) → clean recent number.
- Start the **tokenizer (M1)**: `vocabulary` → `numeric_bucketer` → `categorical/temporal` → `KVTTokenizer`.
- Then race to a **first toy training run** (de-risk "does it train at all").

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
- **Schedule:** baselines **ahead** of plan (real-world OOT bars done early); model (M1+) not started.
- **Risk concentration:** ~80% of technical uncertainty is in the still-unbuilt model + pretraining
  (Phases C–D). Foundations are now strong *and real-world validated* — but they're still foundations.
- **Process:** git/file-handoff friction reduced via heredoc workflow + `git ls-files` verify habit.
