# Progress Log — Credit Foundation Model Framework

**Internal. Not pushed to git.** Dated record of what was actually completed each day.
Legend: ✅ done · 🔄 in progress · ⏭ deferred/blocked.

---

## Day 1 — Thu 18 Jun 2026 (Phase A · Setup)

**Repo & framework scaffold**
- ✅ Public repo live: `github.com/sriramarun/credit-foundation-model` (private), Apache 2.0.
- ✅ Full `credit_fm` framework scaffold pushed (tokenizer / models / data / training /
  inference / evaluation / utils, configs, scripts, notebooks, reference impls, app, tests).
- ✅ `pip install -e .` works; `import credit_fm` OK; CI workflow in place.

**H100 container bring-up**
- ✅ Container created (NGC `pytorch:24.05-py3`), power-user access obtained; repo cloned.
- ✅ GPUs verified through step 3 — **note: only 2 GPUs visible** (`NVIDIA_VISIBLE_DEVICES=0,1`),
  not 8. Looks like a shared multi-participant box.
- ✅ Restart-proof venv approach decided + documented: `/workspace/.venv` with
  `--system-site-packages`; grep-guarded auto-activation; secrets via `/workspace/secrets.env`.
- ✅ Setup + git/dev-workflow captured & published: `docs/container_setup.md` +
  `scripts/setup_container.sh` + CONTRIBUTING. Commits `82af597`, `09f689f`.
- ✅ W&B project set (`credit-foundation-model` via compose env).
- ⏭ **Rolled to Day 2 (not yet done in-container):** create venv + `pip install -e .[dev]`,
  secrets file, git identity/credentials (push-ready), scaffold verify.

**Open / carry-over**
- ⏭ Decision log (`docs/architecture.md` decisions are captured; formal running decision log TBD).

**Next (Day 2 — Fri 19 Jun):** pull `Algoritmica/green-lion-2024-2025` into `data/raw`,
inspect the 71-column schema, and classify fields (static/dynamic, categorical/numeric/
text/temporal) to drive `configs/dutch_mortgages/tokenizer.yaml`.

---

## Day 2 — Fri 19 Jun 2026 (Phase A · Setup finish + data)

**Environment finished in-container**
- ✅ Resolved venv bring-up: `--system-site-packages` venv guarded on `bin/activate` + auto
  `pythonX.Y-venv` install when ensurepip is missing (PRs #1 `fix/setup-venv-guard`,
  #2 `fix/setup-ensurepip`).
- ✅ Scaffold lint cleaned so CI/`ruff` is green (PR #3 `fix/ruff-scaffold-lint`).
- ✅ `setup_container.sh` runs end-to-end: `credit_fm 0.1.0` imports, torch 2.4 + CUDA,
  **8 GPUs visible** (the earlier 2-GPU reading is resolved — full node available).
- ✅ W&B configured (`WANDB_API_KEY` in `/workspace/secrets.env`) and verified.
- ✅ `HF_TOKEN` set; `HF_HOME=/workspace/.hf_cache` for persistent cache.

**Data**
- ✅ Dutch RMBS parquet downloaded: `data/raw/Overall_2024_2025_all_months.parquet`
  (via `hf_hub_download(..., local_dir="data/raw")`). Single-file dataset, not a folder tree.

**Repo committed & verified on `main` (1bd29c3)**
- ✅ `data/` scaffold (README + 2× `.gitkeep`) + `.gitignore` keep-rules — PRs #4, #5.
- ✅ `docs/container_setup.md` fixes: `hf_hub_download` download, secrets block
  (`WANDB_PROJECT`, `HF_HOME`, `chmod 600`, `wandb login --verify`).
- ✅ Verified: parquet is **untracked** (`git ls-files data/` = README + 2 `.gitkeep` only).
- 📝 Note: the secrets/data note landed at the end of `docs/container_setup.md` (174–176)
  rather than `CONTRIBUTING.md`; content correct, CONTRIBUTING already covers secrets. Optional tidy.

**Deferred → Sat 20 Jun** (not done today): profiler + 71-field classification,
`tokenizer.yaml`, `tokenization.md`, `decision_log.md`.

---

## Sat 20 Jun 2026 (Phase A · Data split + Tokenizer kickoff)

Order: **split → classify → build vocab on train only.**

**Done**
- ✅ **Loan-stratified temporal split implemented + verified on the real 767MB file:**
  400k/50k/50k loans (80/10/10), 8.94M/1.12M/1.12M rows, disjoint (no loan_id leakage),
  temporal (train→2021-07, val→2022-07, test→2023-07). `scripts/prepare_data.py` +
  `src/credit_fm/data/splits.py` + `tests/test_data.py` (4 tests pass, ruff clean).
- ✅ **Reviewed real schema:** 71 ESMA cols, 11.18M rows, 500k loans, 24 monthly cutoffs.
  No `origination_date` column; `closing_date` constant (2024-01-01, useless);
  `reporting_date` = 24 cutoffs. **Decision: derive month-precise origination =
  `reporting_date − seasoning_months`** (verified 100% constant per loan, matches
  `origination_year` 100%, range 2008-07→2023-07).
- 🐛 **Found + fixed repo bug:** the whole `src/credit_fm/data/` module (7 files) was never
  committed — the old unanchored `.gitignore` rule `data/` matched `src/credit_fm/data/`
  and dropped it from `git add`. Restored via the `fix/add-data-module` branch. `.gitignore`
  already anchored (`data/*`) so it sticks.

**Merged to main (today)**
- ✅ Data module + split + `docs/decision_log.md` (PRs #6, #7 `fix/add-data-module`).
- ✅ Copyright rebrand **Algoritmica.ai → finevals.ai** (PR #8); 62 files, external HF/GitHub
  refs preserved. main @ `e4a36a1`. ruff clean, 4 tests pass.

**Field classification — DONE & merged (PRs #9–#11, main @ `334a245`)**
- ✅ `classify_fields()` + `find_redundant()` in `schema.py`, driven by `scripts/classify_schema.py`.
- ✅ **Validated against the column glossary: 70/71 static/dynamic match.** The 1 miss is a
  data anomaly (`guarantee_type`: 1 value + 62.8% null vs spec `{NHG,None}`; == `nhg_flag`). DL-010.
- ✅ **Reproducible, not hand-edited:** `find_redundant` auto-detects exact dups + numeric
  `*_bucket`; FD candidates opt-in via `--drop`; `tokenizer.yaml` header records the regen
  command. Fixed two bugs found doing it properly: continuous-column FD false-positives, and
  `arrears_bucket` (a real state) is correctly kept.
- ✅ `tokenizer.yaml` = **42 features** (29 profile / 13 event); 11 constant + 15 redundant
  dropped. CI test `test_find_redundant_flags_duplicate_and_fd` (5 tests pass).

**Smoke test → real baseline (XGBoost, Gate G1)**
- ✅ New extract `all_cutoffs.parquet` (500k loans, 11.27M rows, ~4% default rate). Split is
  robust (400k/50k/50k, temporal, val≈test).
- ✅ Built `scripts/train_baseline.py` — 4 configs on **our temporal split** isolating leakage
  (test ROC / PR-AUC):
  - (1) full, no gate: **0.934 / 0.811** (= the smoke number; leaky)
  - (2) full + gate: 0.839 / 0.435
  - (3) no-leakage, no gate: 0.733 / 0.099
  - **(4) no-leakage + gate = Gate G1: 0.734 / 0.046** (1.7% base rate) ← FM must beat this
- ✅ Key finding: removing contemporaneous-state features drops ROC 0.93→0.73; the
  performing-at-obs gate collapses PR-AUC to 0.046 — the hard, realistic task = FM's room.
- `notebooks/00_smoke_test_splits.ipynb` reframed as config (1) only; points to the real baseline.
- 📝 **Architectural-validation pending** `loan_book.parquet` (`_segment`/`_latent_fragility`) —
  the hidden-latent ceiling that proves XGBoost can't reach the FM's signal. Need that extract.

**Next — tokenizer build (Phase A finish)**
- ⬜ `vocabulary.py` + `base.py` → `numeric_bucketer.py` → `categorical.py`/`temporal.py` →
  `KVTTokenizer` (encode/decode, roundtrip ≥99%). Vocab fit **on `train` only**. → M1.

### Split methodology (LOCKED — capture in docs/decision_log.md)
- **Split by `loan_id`, never by row.** All 24 cutoffs of a loan go to one split (else
  loan-level leakage: same loan, ±1 month, in train and test → fake test performance).
- **Split by origination date, train < val < test in time** (loan-stratified *temporal*),
  80/10/10. Matches production use (today's model scores tomorrow's loans).
- **Label-horizon leakage is handled at the label-generator layer, not the split** — e.g.
  `default_within_6m` requires the prediction cutoff ≥6 months before panel end.
- **Tokenizer vocab + numeric bin edges fit on `train` only** (the hard rule).
- **Audit trail** `splits.meta.json`: seed=42, split_date, source_panel_sha256,
  per-split loan counts, split_criterion, train/val/test origination ranges, code_commit.
- For MLM pretraining you *may* combine train+val (objective is label-free); keep `val` for
  memorization monitoring (PRAGMA's synthetic-data risk) + downstream validation. Vocab still train-only.

---

<!-- Append one section per working day below. -->
