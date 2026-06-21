# Project Plan — Credit Foundation Model Framework

**Internal. Do not push to git.**

## Objective
Build an open-source (Apache 2.0) framework for training **encoder-only (PRAGMA-style, MLM)**
credit foundation models on tabular credit panels, plus **two reference implementations**
(Dutch mortgages, invoice financing). Engagement: Algoritmica.ai × Sriram Krishnan, sponsored
by NVIDIA (8× H100), 3-month delivery.

## Timeline
~12 weeks (≈60 working days). Kickoff **Thu 18 Jun 2026**; target handoff window **early Sep
2026**. Phase targets below are windows, not fixed dates — actuals drift, so track live status
in [`PROGRESS.md`](PROGRESS.md) (dated log) and [`TODO.md`](TODO.md) (phase backlog).

**Status (as of Sat 20 Jun 2026):** Phase A in progress. Repo + framework scaffold live; H100
container up (8 GPUs); dataset downloaded; **loan-stratified temporal split done & verified**;
missing `src/credit_fm/data/` module restored. Next: field classification + tokenizer.

## Phases & milestones

| Phase | Weeks | Days | Focus | Milestone / Gate |
|-------|-------|------|-------|------------------|
| A | 1–2  | 1–10  | Setup, CI, key-value-time tokenizer | **M1** tokenizer done |
| B | 3–4  | 11–20 | Data layer + Dutch configs + baselines | **G1** trustworthy labels + strong baseline |
| C | 5–6  | 21–30 | Three-branch model + toy validation | **M2** model forward + toy train |
| D | 7–8  | 31–40 | Training pipeline + full pretraining | **M3** pretrained 30M checkpoint |
| E | 9–10 | 41–50 | Inference, evaluation, dashboard | **M4** Dutch mortgages reference complete |
| F | 11–12| 51–60 | Invoice reference + docs + demo + handoff | **M5** invoice ref · **M6** handoff |

## Deliverables (OSS release manifest)

17 components across **Code / Data / Documentation** (the engagement's deliverable taxonomy).
Full status + repo locations in the internal [`DELIVERABLES.md`](DELIVERABLES.md) (not pushed).
Maps to phases:

| Category | Items | Produced in |
|----------|-------|-------------|
| **Code** | Evaluation · Preprocessing · Model Architecture · Libraries & Tools · Training · Inference | Libs ✅ (A); Preproc 🟡 (A/B); Eval 🟡 (B); Arch (C); Training (D); Inference (E) |
| **Data** | Datasets · Evaluation Data · Sample Outputs · Model Weights · Model Metadata · Configuration | Datasets ✅; Eval Data ✅ (B); Config 🟡 (A); Weights/Metadata/Outputs (D/E) |
| **Docs** | Data Card · Research Paper · Evaluation Results · Model Card · Technical Report | Eval Results 🟡 (B, baseline); cards 🟡 templates; paper/report (F) |

Legend: ✅ done · 🟡 partial/template · (blank) = scaffold/planned in the named phase.

## Key design decisions (locked — see SPEC_SUMMARY)
1. Encoder-only + MLM (not decoder/causal).
2. Three-branch encoders: Profile State (3L) + Event (4–5L) + History (4–6L).
3. Key-value-time disentangled tokenization.
4. 30M default (Chinchilla-honest on ~600M synthetic tokens); 50M only with extra public data.
5. Apache 2.0 from day one.
6. HuggingFace primary; NeMo optional (`--backend`).

## Critical path
Tokenizer (A) → Data + baseline gate (B/G1) → Model (C) → **Pretraining run (D)** is the
longest-pole / highest-risk item; protect weeks 7–8. Everything downstream (E) depends on a
converged checkpoint. Invoice reference (F) depends on external data arriving by **Day 45**.

## Risks
| Risk | Mitigation |
|------|------------|
| Memorization — 30M on ~600M synthetic tokens | Hold to 30M; only go 50M with supplementary public data; watch val loss |
| Weak baseline makes lift misleading | Strong XGBoost/LightGBM + calibration; gate G1 before pretraining |
| Data leakage (future-state fields) | Observation-date discipline; leakage review; loan-stratified temporal splits |
| Invoice data slips past Day 45 | Escalate to Algoritmica early (week 1); keep Dutch ref as the primary deliverable |
| H100 access/storage limits | Checkpoint frequently; size the pretraining run to the GPUs actually available |
| AML downstream (PRAGMA fails) | Document as known limitation; do **not** attempt |

## Open questions (track to closure)
1. Invoice financing data source + schema — needed by **Day 45 (week 9)**.
2. OSS release timing — full at end vs staged (checkpoint first)?
3. AML as a downstream task — recommend documenting as out-of-scope limitation.
4. Engagement continuation beyond month 3 (maintenance + more references).

## Existing assets (do not rebuild)
- `Algoritmica/green-lion-2024-2025` synthetic Dutch RMBS (500k×24).
- deeploans synthetic-data-designer (regenerate at scale if needed).
- 53-test SQL validation suite (reuse in `prepare_data.py`).
- NVIDIA TFM blueprint (adapt patterns; don't fork).
- AnushaFreshStart/A-foundation-credit-model FastAPI skeleton (lift UI).

## Tracking
W&B project `credit-foundation-model`; every run logs model size, dataset, hyperparameters,
git commit, compute hours. Update [`TODO.md`](TODO.md) daily; review milestones at each phase end.
