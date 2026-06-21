# Weekly Progress — Credit Foundation Model Framework

**Period:** Week 1 (18–21 Jun 2026)  ·  **Phase:** A — Setup + Tokenizer  ·  **Overall: 🟢 On track**

> Internal status report (shareable summary). Not part of the OSS repo.

Open-source (Apache 2.0) framework for training credit foundation models, plus Dutch-mortgages
and invoice-financing reference implementations. Engagement: finevals.ai × Sriram Krishnan,
NVIDIA-sponsored (8× H100), ~12-week delivery → target handoff **early Sep 2026**.

## Headline
The foundation is in place — repository, a reproducible/leakage-safe data pipeline, and an
**honest baseline**. The model build starts next.

## Done this week
- **Infrastructure** — 8× H100 container fully set up (reproducible env, experiment tracking,
  dataset access); private repo live with CI.
- **Data pipeline** — loan-stratified **temporal** train/val/test split (leakage-safe, audited)
  on the 500k-loan / 11.3M-row synthetic Dutch RMBS panel.
- **Schema** — all 71 ESMA fields classified into model inputs **reproducibly** (validated
  70/71 against the data glossary) → a 42-feature tokenizer configuration.
- **Baseline (Gate G1)** — XGBoost baseline established (see below).

## Key result — Baseline (Gate G1)
The bar the foundation model must beat, on the realistic task: predict *new* defaults among
loans that are **currently performing**, using non-state features only.

| Metric | Value |
|--------|-------|
| ROC-AUC | **0.73** |
| PR-AUC  | **0.046** (1.7% default base rate) |

A leakage-/state-inflated version scores 0.93 — we deliberately report the honest, harder
number. The low PR-AUC is precisely the headroom a sequence-based foundation model can exploit.

## In progress / next
- **Tokenizer build** (key-value-time; vocabulary fit on train only) → **Milestone M1**.
- Then: three-branch encoder model → pretraining on 8× H100.

## Risks & open items
- **Latent-segment data extract** (`loan_book` with `_segment`/`_latent_fragility`) needed to
  run the validation that demonstrates the tabular-model ceiling — the core "why a foundation
  model" evidence. *(action: data team)*
- **Experiment-tracking hosting** (hosted vs self-hosted/offline) — decide before pretraining,
  given the sovereign-cloud / data-residency positioning. *(DL-009, open)*
- **Invoice-financing dataset** — needed by ~week 9. *(action: Algoritmica)*

## Deliverables (17-component release taxonomy)
**3 done** (Libraries & Tools, Datasets, Evaluation Data) · **6 in progress** (Preprocessing,
Evaluation, Configuration, Model Metadata, Data Card, Model Card) · **8 planned**.

## Timeline
Phase A (of A–F) wrapping up; on track for the ~12-week window — no schedule slippage.
