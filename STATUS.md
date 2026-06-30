# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 27 Jun 2026)*

**📌 28 Jun — M2 DONE + data-scale finding (DL-015).** Full hierarchical 3-branch model
(`CreditFoundationModel`, 25.5M @ dim=384) + AdamW-cosine `train_mlm` (now with dropout + best-val
checkpointing) + `pretrain.py`. Trains on real Fannie (toy: 6.38→0.10; H100/bf16). Architecture FROZEN.
**Diagnostic (100k loans, val split):** model overfits regardless of size/dropout — train→~0.1, val
plateaus **~2.6–2.8 then rises** (25.5M, 25.5M+dropout, and 1.4M all the same). 100k ≈ 25M tokens is
~20× under the Chinchilla budget for 25.5M params (DL-004). **Conclusion: the lever is DATA SCALE,
not model size/regularisation** → parallel-encode the full corpus, train on ~2M loans; **gate the FM
on downstream OOT (ROC 0.757), not MLM loss.** **Next: M3 (parallel encoder → full-corpus pretrain).**

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
| **Overall** | 🟢 Well ahead of plan — M1 (tokenizer) + M2 (model trains on real data) both done by 28 Jun |
| **Phase** | A + B + **C/M2 done** (model trains); next is **Phase D / M3** — pretrain at scale (week ~2 of ~12) |
| **Next milestone** | **M3** pretrained 30M checkpoint — **internal aim ~18–21 Jul** (~3 wks; official target 12 Aug, ~3–4 wks early) |
| **Key metric (the bar)** | **Fannie OOT crisis: ROC 0.757 / PR-AUC 0.024** (real-world, leakage-free). Recent (2023–24): ROC ~0.78. |
| **Biggest unknown** | Will the FM **generalize and beat ~0.76 OOT** once trained on ~2M loans? (the M3→E verdict, DL-015) |
| **Planning horizon** | Schedule laid out **to 14 Aug** (see Timeplan below): M3 checkpoint ~18 Jul → downstream verdict by ~1 Aug → report by 14 Aug |

## Milestone progress

```
M1 tokenizer      ▓▓▓▓▓▓▓▓▓▓  DONE — fit on real Fannie train (25.6M rows) → 440-token vocab; anchored/per-field bins + cal= macro token; 100% roundtrip, 0% OOV; merged (PR #31/#32), tests 14/14  (done 26 Jun, ahead of 1 Jul)
G1 baseline gate  ▓▓▓▓▓▓▓▓▓▓  DONE — real-world OOT bars established (crisis + recent)
M2 model          ▓▓▓▓▓▓▓▓▓▓  DONE (28 Jun, ahead of 29 Jul) — data layer + hierarchical model (25.5M @ dim384) + train loop; **toy run on REAL Fannie (2000 loans, H100, bf16): loss 6.38→0.10 over 300 steps, checkpoint to GCS.** arch FROZEN. 62 tests green.
M3 pretrained     ▓▓▓▓▓▓▓▓▓░  FIRST CHECKPOINT DONE (29 Jun, ~6 wks early) — full corpus 1.24M loans / 453M tokens, batch128/12k steps. **GENERALIZES: val MLM 0.31 vs 100k's ~2.7** (train 0.074). Checkpoint m3_full.pt on GCS. Refinements optional (more steps / multi-vintage / tokenizer v2).
M4 / Phase E      ░░░░░░░░░░  the verdict — FM embeddings vs ROC 0.757 (downstream eval)   aim ~26 Jul–1 Aug
report + cards    ░░░░░░░░░░  evaluation report + model card   aim ~9–14 Aug (planning horizon)
M5/M6 handoff     ░░░░░░░░░░  invoice ref + final handoff   target 9 Sep
```

## Timeplan to 14 Aug (today = 28 Jun; we are ~3–4 wks ahead of the original phase plan)

| Window | Focus | Outcome |
|---|---|---|
| **28 Jun – 4 Jul** | Parallel encoder (`encode_dataset.py --workers`); W&B decision (DL-009); tokenizer-v2 decision | encode unblocked |
| **5 – 11 Jul** | Full-corpus encode (~2M loans); lazy/streaming dataset; first at-scale pretrain sanity | data + pipeline ready |
| **12 – 18 Jul** | **Full pretraining run(s)**; iterate; watch train-vs-val converge | **→ M3 checkpoint ~18 Jul** |
| **19 – 25 Jul** | Freeze 30M + training report; wire Phase E (`extract_embeddings`, `evaluate_downstream`) | M3 done; eval harness ready |
| **26 Jul – 1 Aug** | **Downstream eval: FM embeddings vs ROC 0.757** | **the real verdict** |
| **2 – 8 Aug** | Iterate on the result (more data / macro features / longer pretrain if needed) | tuned result |
| **9 – 14 Aug** | Evaluation report + Fannie model card; buffer | deliverable for 14 Aug |

**M3 duration = ~3 weeks of work** (parallel encode is the hard blocker; pretraining convergence is the
variable). Producing *a* checkpoint is ~3 wks; whether it **beats 0.757** is the Phase-E verdict (DL-015).

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

**Now / next — M3 (data scale)**
- [x] **Parallel encoder** (spawn-based `--workers`) — 17× speedup.
- [x] **Full TRAIN corpus encoded (29 Jun): 1.24M loans → 453M tokens, 63 shards, ~27 min.**
  453M tokens ≈ Chinchilla-matched for 25.5M → escapes the 100k overfitting. Val (155k) already encoded.
- [ ] **Lazy/streaming dataset** — current `CreditSequenceDataset` loads all shards into RAM (~20GB);
  stream per-shard so the full corpus needn't fit in memory. **Next brick.**
- [ ] **Full-corpus pretrain** of 25.5M on 1.24M loans (best-val + dropout on) — the run that should
  finally generalize. Watch train-vs-val *converge* (vs the 100k plateau).
- [ ] **Downstream eval** — `[USR]` embeddings → OOT harness → **FM-vs-0.757** (the real gate, DL-015).
- Optional: tokenizer v2 (multi-vintage incl. 2006–2010) for crisis coverage; W&B (DL-009); multi-GPU.

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
