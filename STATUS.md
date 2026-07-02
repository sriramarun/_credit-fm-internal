# Status — Credit Foundation Model Framework

> Internal · the **live dashboard + weekly report**. Refresh every week and when a milestone
> lands. The stable roadmap is in [`PLAN.md`](PLAN.md). *(Last updated: 2 Jul 2026)*
> **Tracker of record is now the xlsx** (`Open source credit foundation model - Project Manager.xlsx`,
> this folder) + the OS-components PDF as deliverables north star; these MD files are narrative backup.

**📌 2 Jul — FINE-TUNE VERDICT (FM beats the bar) + config-driven framework (PRs #53/#54).**
(a) **Phase E complete.** Frozen probe lost as expected (emb-XGB 0.687 / combined 0.719 / linear 0.676
vs features 0.746 — the handicap match). **Fine-tune ladder flips it: frozen-head 0.7192, full 0.7487,
LoRA r=8 0.7526 > 0.746** (12mo default, Dec-2016 cutoff, 45k held-out loans). Matches PRAGMA's
published pattern (LoRA ≥ task-specific model). Caveats: +0.007 is thin (~190 test positives); PR-AUC
still below features (0.0127 vs 0.0171). **Crisis-OOT (0.757) is the remaining verdict.**
(b) **Whole repo is config-driven** — blueprint parity: config engine (`utils/config.py`: include /
${a.b} interpolation / dotted overrides / lineage) + 12 recipes in `configs/fannie_mae/`; all 13
scripts one grammar (`-c recipe --key.path value`). Whole pipeline retargets to the crisis panel via
`--run_name run_2008_2010`. 70 tests green.
(c) **Anchors set:** NVIDIA blueprint = baseline, PRAGMA = improvement target; goal = the FRAMEWORK,
not score-chasing. **Multi-GPU DDP deliberately sequenced last.** Next: release items (model/data
cards, notebooks, weights publish) + crisis-OOT.

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
| **Overall** | 🟢 ~5 wks ahead — M1+M2+M3+Phase E all done by 2 Jul; **LoRA fine-tune beats the recent-window bar (0.7526 > 0.746)**; framework fully config-driven |
| **Phase** | E done (fine-tune verdict in) → now **release hardening** (cards, notebooks, weights) + **crisis-OOT**; DDP last |
| **Next milestone** | **Crisis-OOT verdict** (needs 2008–10 processed panel) + OS release package per component map |
| **Key metric (the bar)** | Recent-window: **beaten** (LoRA 0.7526 vs features 0.746). **Fannie OOT crisis ROC 0.757** still open. PR-AUC gap open (0.0127 vs 0.0171). |
| **Biggest unknown** | Does the LoRA win **replicate on crisis-OOT** (2008–10)? Recent-window margin is thin (~190 test positives) |
| **Planning horizon** | Schedule laid out **to 14 Aug** (see Timeplan below): M3 checkpoint ~18 Jul → downstream verdict by ~1 Aug → report by 14 Aug |

## Milestone progress

```
M1 tokenizer      ▓▓▓▓▓▓▓▓▓▓  DONE — fit on real Fannie train (25.6M rows) → 440-token vocab; anchored/per-field bins + cal= macro token; 100% roundtrip, 0% OOV; merged (PR #31/#32), tests 14/14  (done 26 Jun, ahead of 1 Jul)
G1 baseline gate  ▓▓▓▓▓▓▓▓▓▓  DONE — real-world OOT bars established (crisis + recent)
M2 model          ▓▓▓▓▓▓▓▓▓▓  DONE (28 Jun, ahead of 29 Jul) — data layer + hierarchical model (25.5M @ dim384) + train loop; **toy run on REAL Fannie (2000 loans, H100, bf16): loss 6.38→0.10 over 300 steps, checkpoint to GCS.** arch FROZEN. 62 tests green.
M3 pretrained     ▓▓▓▓▓▓▓▓▓░  FIRST CHECKPOINT DONE (29 Jun, ~6 wks early) — full corpus 1.24M loans / 453M tokens, batch128/12k steps. **GENERALIZES: val MLM 0.31 vs 100k's ~2.7** (train 0.074). Checkpoint m3_full.pt on GCS. Refinements optional (more steps / multi-vintage / tokenizer v2).
M4 / Phase E      ▓▓▓▓▓▓▓▓▓░  RECENT-WINDOW VERDICT DONE (2 Jul, ~4 wks early) — frozen probe loses (0.687/0.719 vs 0.746) but **LoRA fine-tune WINS: 0.7526 > 0.746** (full 0.7487, frozen-head 0.7192). Crisis-OOT (0.757) is the open final leg.
framework/config  ▓▓▓▓▓▓▓▓▓▓  DONE (2 Jul, PRs #53/#54) — config engine + 12 recipes; all 13 scripts `-c recipe + --key.path` (blueprint parity); lineage in every artifact
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
- [x] **Full-corpus pretrain DONE (29 Jun): 25.5M on 1.24M loans, batch 128, 12k steps (~1.25 epochs).**
  **GENERALIZES — train 0.074 / val 0.31** (vs 100k's val ~2.7 that rose). Checkpoint `m3_full.pt` on GCS.
  (In-RAM dataset worked; batch 512 OOM'd → 128 — dense O(L²) attention. Length-bucket/flash-attn deferred.)
- [x] **Downstream probe #1 (frozen embeddings) — 29 Jun.** Dec-2016 obs → 12-mo default, 150k train loans,
  loan-holdout. **features-XGB 0.746 ROC | FM-emb 0.687 | combined 0.719 | linear-probe 0.676.**
  → **FM frozen embeddings LOSE to features** (but 0.69 ≫ 0.5 = real signal). Expected for a frozen
  *unsupervised* emb vs *supervised* XGB on the same fields, on short benign in-sample history.
- [ ] **★ Fine-tune / LoRA the FM on the default task** (the real FM test — frozen probe is the handicap
  match; adapt pretrained weights + `classification_head`, compare to 0.746). **Next build.**
- [ ] Crisis-OOT verdict (2008–10, bar 0.757) where sequence/regime signal should matter most — needs
  the crisis-era processed panel (option B).
- [ ] Lazy/streaming dataset (only needed for corpora bigger than RAM); tokenizer v2 (multi-vintage,
  incl. 2006–2010 for crisis coverage) + longer pretrain — refinements after the first verdict.
- Optional: W&B (DL-009); multi-GPU; length-bucketed batching + flash-attn (to lift batch size).

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
