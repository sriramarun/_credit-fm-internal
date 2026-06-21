# Spec Summary — Credit Foundation Model Framework

**Internal reference.** Condensed from the engagement specification (authoritative doc lives
with the engagement). Do not push.

## What's being built (build order)
1. **`credit_fm` framework** — configurable, schema-agnostic, encoder-only credit FM training.
2. **Dutch mortgages reference** — 500k synthetic RMBS (ESMA Annex 2), 30–50M checkpoint.
3. **Invoice financing reference** — second asset class, data TBD (Algoritmica).

## Architecture
- **Tokenization (key-value-time):** each field → semantic-type key token (~70), value
  token(s) (numeric→16–32 percentile buckets, categorical→single, text→BPE), temporal
  coordinate (`8*ln(1+s/8)` + cyclical hour/day/week sin·cos). Value vocab 200–500.
- **Three-branch encoder:** Profile State (3L, static + lifelong events) → [USR];
  Event (4–5L, per-event independent, +calendar features) → [EVT]; History (4–6L,
  [USR]+[EVT_1..n] + time-to-last RoPE) → final embeddings. 30–100M params.
- **Pretraining:** MLM, three masking sources — 15% tokens / 10% events / 10% semantic types.
- **Downstream:** embedding probe (frozen + linear/XGBoost) or LoRA (rank 8, α 8, QKV+MLP).

## Locked decisions + rationale
1. **Encoder-only + MLM** — PRAGMA +130% PR-AUC on discriminative credit tasks.
2. **Three-branch** — PRAGMA's Profile State Encoder gave +31.8% PR-AUC.
3. **Key-value-time** — preserves field identity ("LTV 85" ≠ "DPD 85").
4. **30M default** — Chinchilla floor 20 tok/param; ~600M synthetic tokens caps honest size.
5. **Apache 2.0** — differentiation via services/reference packs, not code secrecy.
6. **HF primary, NeMo optional** — NVIDIA sponsor path works without locking users in.

## Standards
- Python 3.10+, `ruff`, type hints on public APIs, Google docstrings.
- Test per module: tokenizer roundtrip >99%; model shapes; split no-leak; metrics ≈ sklearn 1e-6;
  `test_e2e` 100 loans × 10 steps < 60s non-NaN. Framework coverage target >80%.
- Every run in W&B (`credit-foundation-model`): size, dataset, hyperparams, commit, compute hrs.
- Every script `--seed`, logs seed + commit. Semantic versioning; framework versions separately
  from reference checkpoints.

## Deliverables
- **Framework:** `pip install -e .` package, `docs/`, notebooks, test suite >80%, CI.
- **Per reference:** checkpoint (LFS), configs, `train.sh`/`evaluate.sh`, model card, data card,
  evaluation report (ROC-AUC/PR-AUC/KS/Gini/Brier/lift; baseline / emb-only / combined / LoRA).
- **Handoff:** FastAPI dashboard, 5-min demo video, final technical report + v2 roadmap.

## Differentiation
Open-source (vs proprietary PRAGMA) · regulator-aligned ESMA Annex 2 schema ·
sovereign-cloud-deployable (no external APIs; GCC data-residency).

## References
PRAGMA (Ostroukhov 2026, arXiv:2604.08649) · NVIDIA TFM blueprint · BERT (Devlin 2019) ·
LoRA (Hu 2021) · Chinchilla (Hoffmann 2022) · ESMA Annex 2 (EU 2020/1224).
