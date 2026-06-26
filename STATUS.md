# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 26 Jun 2026)*

**📌 26 Jun — M1 tokenizer DONE.** KVT tokenizer fit on the real Fannie train split (25.6M rows) →
**440-token** vocab; re-fit with reviewer #1/#2 incorporated (threshold-anchored + per-field bins;
`cal=<YYYYQ#>` macro-regime token). 100% lossless roundtrip, 0% OOV; merged to `main` (PR #31 code,
PR #32 artifacts), tests 14/14. **Next is M2** — the three-branch model (data layer first).
⚠️ Repo docs (`docs/tokenization.md`, `docs/decision_log.md`) still need updating for the new tokens.

**📌 25 Jun — real-world Fannie baselines done; the bar for the FM is set.** The full 26-yr Fannie
book is ingested to GCS, and two honest **out-of-time (OOT)** baselines are built. Baseline phase
is essentially complete; the model itself (tokenizer → encoders → pretrain) is the critical path now.

## At a glance

| | |
|---|---|
| **Overall** | 🟢 On track (baselines + tokenizer done ahead of plan; model not started) |
| **Phase** | A done (tokenizer) + B done (baselines); next is Phase C — the model (week ~2 of ~12) |
| **Next milestone** | **M2** model forward + toy train — target **29 Jul 2026** (M1 tokenizer DONE 26 Jun, ahead of 1 Jul) |
| **Key metric (the bar)** | **Fannie OOT crisis: ROC 0.757 / PR-AUC 0.024** (real-world, leakage-free). Recent (2023–24): ROC ~0.78. |
| **Biggest unknown** | Will the FM train *and* beat ~0.76 OOT? (no model code exists yet) |

## Milestone progress

```
M1 tokenizer      ▓▓▓▓▓▓▓▓▓▓  DONE — fit on real Fannie train (25.6M rows) → 440-token vocab; anchored/per-field bins + cal= macro token; 100% roundtrip, 0% OOV; merged (PR #31/#32), tests 14/14  (done 26 Jun, ahead of 1 Jul)
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

**Done (week ~2)**
- **M1 tokenizer COMPLETE** — `vocabulary`/`numeric_bucketer`/`categorical`/`KVTTokenizer` compose;
  fit on real Fannie train (25.6M rows) → 440-token vocab; reviewer #1 (anchored + per-field bins)
  and #2 (`cal=<YYYYQ#>` macro token) shipped; `tokenizer.json` + QA report committed.

**Now / next**
- **Update repo docs** for the new tokens: `docs/tokenization.md` (fused `field=value`, `t=` + `cal=`,
  anchored bins — current text describes a stale log-seconds/BPE design) and `docs/decision_log.md`
  (add DL-011 calendar/macro token, DL-012 anchored/per-field bins).
- Start **M2**: data layer (`dataset.py`/`collators.py` + MLM masking 15/10/10) → three-branch model.
  Size context window at **1024** (full-corpus loans hit `max_events=60` ≈ ~1000 tokens).
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
- **Schedule:** baselines **ahead** of plan (real-world OOT bars done early); model (M1+) not started.
- **Risk concentration:** ~80% of technical uncertainty is in the still-unbuilt model + pretraining
  (Phases C–D). Foundations are now strong *and real-world validated* — but they're still foundations.
- **Process:** git/file-handoff friction reduced via heredoc workflow + `git ls-files` verify habit.
