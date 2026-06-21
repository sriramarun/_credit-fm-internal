# Credit FM — Internal Planning (DO NOT PUSH)

This folder is **internal planning only**. It lives *outside* the git repo
(`../credit-foundation-model`) and must **never** be committed or pushed to GitHub.

## Contents (reset 21 Jun 2026 — two surfaces, by purpose)
- [`STATUS.md`](STATUS.md) — **the now**: live dashboard + weekly report (RAG, milestones, this/next week, key metric, blockers). *Start here.*
- [`PLAN.md`](PLAN.md) — **the map**: milestones with target dates + full phase-by-phase task tracker; risks; open questions.
- [`DELIVERABLES.md`](DELIVERABLES.md) — 17-component release manifest + status.
- [`SPEC_SUMMARY.md`](SPEC_SUMMARY.md) — condensed reference of the engagement spec.
- `_archive_trackers/` — the previous overlapping trackers (PROJECT_PLAN, TODO, PROGRESS, weekly_progress), kept for history.
- `_archive_old_repo/` — superseded artifacts from the original decoder-only scaffold.

## How to use it
- **Daily / weekly:** update `STATUS.md` (move items Done → This → Next; refresh blockers + the "Last updated" date).
- **When a milestone lands or scope changes:** update `PLAN.md` (check tasks, flip milestone status).
- **Rule:** `PLAN.md` = stable map (changes rarely); `STATUS.md` = volatile now (changes weekly).
  Don't reintroduce a third status surface — that's what caused the previous drift.

## Why the reset
Four files (PROJECT_PLAN / TODO / PROGRESS / weekly_progress) all conveyed status, overlapped,
and drifted — "where are we?" wasn't answerable at a glance. Now: one dashboard (`STATUS`), one
map (`PLAN`). Dates live at the **milestone level** (6 stable targets), not daily tasks (those
drift — the rigid `60_DAY_SCHEDULE` was retired for the same reason).

## Timeline
~12 weeks. Kickoff **Thu 18 Jun 2026** → handoff **~9 Sep 2026** (6 phases × 2 weeks; see `PLAN.md`).
