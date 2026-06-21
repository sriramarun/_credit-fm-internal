# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 21 Jun 2026)*

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

## Key result — the story in one slide

> Among loans that look healthy today, predicting *new* defaults is hard for tabular models
> (PR-AUC 0.046). The reason: a hidden borrower-fragility segment causes **16–32× more defaults**
> but isn't visible in standard credit fields. The foundation model reads each loan's behavioural
> sequence to recover that latent — that's the headroom above the 0.73 baseline.

## Blockers & asks
- 🔴 **Latent-segment extract** onto the container (`loan_book.parquet`) — to run the ceiling
  validation as a committed artifact. *(have the file locally; needs transfer)*
- 🟡 **W&B hosting decision** (hosted vs offline/self-hosted) before pretraining. *(DL-009)*
- 🟡 **Invoice-financing dataset** — needed by ~week 9. *(Algoritmica)*

## Health notes
- **Schedule:** on track; baseline is slightly *ahead* of plan, tokenizer (M1) not yet hit.
- **Risk concentration:** ~80% of technical uncertainty is in the still-unbuilt model + the
  pretraining run (Phases C–D). Foundations are strong but they're foundations.
- **Process:** week 1 had notable git/setup friction (now documented in `CLAUDE.md` + setup script).
