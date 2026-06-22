# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 22 Jun 2026)*

**📌 22 Jun — primary corpus switched to Fannie Mae** (real-world US single-family fixed,
~25 yrs, ~100 quarterly parquet snapshots in GCS). Dutch synthetic panel → validation/ablation
(keeps `_segment` ceiling proof). Scaffolding landed (`ingest_fannie_mae.py`, `configs/fannie_mae/`,
`docs/data/fannie_mae.md`); next: GCS auth on container + dev-sample ingest → Fannie Gate-G1.

## At a glance

| | |
|---|---|
| **Overall** | 🟢 On track |
| **Phase** | A — Setup + Tokenizer (week 1 of ~12) |
| **Next milestone** | **M1** tokenizer — target **1 Jul 2026** |
| **Key metric (Gate G1)** | XGBoost baseline **ROC-AUC 0.73 / PR-AUC 0.046** — the bar the FM must beat |
| **Biggest unknown** | Will the foundation model train & beat 0.73? (none of the model exists yet) |

## Milestone progress

```
M1 tokenizer      ▓▓▓▓▓▓▓░░░  ~70%   target 1 Jul
G1 baseline gate  ▓▓▓▓▓▓▓▓░░  baseline done early; gate review   target 15 Jul
M2 model          ░░░░░░░░░░   not started   target 29 Jul
M3 pretrained     ░░░░░░░░░░   not started   target 12 Aug
M4 Dutch ref      ░░░░░░░░░░   not started   target 26 Aug
M5/M6 handoff     ░░░░░░░░░░   not started   target  9 Sep
```

## This week (cadence: Last / This / Next)

**Done so far (week 1)**
- 8× H100 container fully set up; private repo + CI live.
- Leakage-safe **temporal** train/val/test split (audited) on the 500k-loan / 11.3M-row panel.
- Reproducible 71-field classification → 42-feature tokenizer config (validated 70/71 vs glossary).
- **Honest baseline established** — Gate G1 = ROC 0.73 / PR-AUC 0.046.
- **Architectural proof located** — hidden `_segment` latent drives a **16–32× default spread**
  invisible to ESMA-feature models (the XGBoost ceiling the FM is built to break).

**Now / next**
- Build the **tokenizer** (`vocabulary` → `KVTTokenizer`, vocab on `train` only) → **M1**.
- Then race to a **first toy pretraining run** (de-risk "does it train at all" early).

## Key result — the story in one slide (now quantified)

> Among currently-performing loans, predicting *new* defaults is hard for tabular models
> (Gate G1 PR-AUC **0.046**). The reason: a hidden borrower-fragility **segment** drives a **34×**
> default spread (0.28% → 9.60%) but is **invisible to standard credit fields** — XGBoost recovers
> it at only 65% vs a 63% majority baseline. If a model *could* see the segment, PR-AUC nearly
> **doubles (0.046 → 0.090, +95%)** and ROC-AUC +0.11. The foundation model reads each loan's
> behavioural sequence to recover that latent — **that +95% is the headroom it's chasing.**

## Blockers & asks
- 🟢 Ceiling validation **built** (`train_baseline.py --book` → 4-config table + segment proof in
  `reports/baseline_report.md`). Minor: get `loan_book.parquet` onto the container to regenerate there.
- 🟡 **W&B hosting decision** (hosted vs offline/self-hosted) before pretraining. *(DL-009)*
- 🟡 **Invoice-financing dataset** — needed by ~week 9. *(Algoritmica)*

## Health notes
- **Schedule:** on track; baseline is slightly *ahead* of plan, tokenizer (M1) not yet hit.
- **Risk concentration:** ~80% of technical uncertainty is in the still-unbuilt model + the
  pretraining run (Phases C–D). Foundations are strong but they're foundations.
- **Process:** week 1 had notable git/setup friction (now documented in `CLAUDE.md` + setup script).
