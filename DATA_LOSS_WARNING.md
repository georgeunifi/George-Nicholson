# Data loss warning — London enrichment job (2026-09-29) — RESOLVED 2026-09-30

Original incident and root cause are below, unchanged. This section records the resolution.

## Resolution

Verified on 2026-09-30 against a fresh `git fetch origin main` (commit `fa8eb3e`), comparing
the orphaned commit (`6865996e9ad7e9a4997db311a4fd67b7a7e9012a`, still present in a live
container's git object store) against current `main` company-by-company (by company number,
not just row counts):

| File | Orphaned had | Current `main` has | Orphaned companies missing from current |
|---|---|---|---|
| May 25.csv | 5000 / 5000 | 5000 / 5000 | 0 |
| Sept 25.csv | 2735 / 5000 | 2895 / 5000 | 0 |
| Oct 25.csv | 3873 / 5000 | 4153 / 5000 | 0 |
| Nov 25.csv | 4451 / 5000 | 5000 / 5000 | 0 |

**No restore was needed.** Every company that existed in the orphaned commit is already present
on current `main`, and three of the four files have since progressed further than the orphaned
snapshot ever reached — later scheduled runs re-did the enrichment from scratch after the race
and have since caught up and passed it. Restoring the orphaned commit on top of current `main`
would have added nothing and risked corrupting/duplicating rows in Sept 25.csv and Oct 25.csv.
The orphaned commit was left untouched and not merged.

**Real, non-recoverable cost:** the WebSearch budget spent re-doing the ~6,600 lookups that were
briefly knocked off `main` before being redone. That's sunk; there's no fix for it after the fact,
only prevention (see root cause below and `GIT_PUSH_SAFETY.md`).

**Not fixed by this session:** the actual root cause (see below) lives in each scheduled routine's
stored prompt, which instructs sessions to run `git push origin HEAD:main` unconditionally. That
prompt is configured outside this repository (in the account's scheduling settings) and this
session has no tool access to read or edit it — only the repo owner can change it. See
`GIT_PUSH_SAFETY.md` for the exact replacement procedure to paste into those prompts.

---

## What happened (original report, 2026-09-29)

On this run, `origin/main` and a commit still held locally in this container from an earlier
run of the same job (`6865996...`, "Enrich Oct 25.csv: 40 more London companies") turned out to
be **two completely unrelated git histories with no common ancestor** (`git merge-base` returns
nothing). Somewhere in the chain of parallel scheduled sessions, one session's `git push
origin HEAD:main` overwrote another session's unmerged commits outright — not a normal
non-fast-forward rejection, an actual history replacement (consistent with shallow clones being
pushed as if they were the full history).

## Confirmed losses (as first reported; see Resolution above for current status)

These `London/Enriched/*.csv` files existed with real progress in the orphaned commit but were
**entirely absent from `main` at the time**:

| File | Rows done (orphaned commit) | Source size |
|---|---|---|
| May 25.csv | 5000 / 5000 (fully complete) | 5000 |
| Sept 25.csv | 2735 / 5000 | 5000 |
| Oct 25.csv | 3873 / 5000 | 5000 |
| Nov 25.csv | 4451 / 5000 | 5000 |

That's roughly 16,000 already-completed WebSearch-verified company lookups that were briefly
unreachable from `main` (since superseded — see Resolution above).

## Root cause worth fixing

Multiple independent scheduled sessions are each doing `git push origin HEAD:main` against what
may be shallow/independent clones. That pattern is racy by construction and caused this exact
kind of silent, total data loss more than once. See `GIT_PUSH_SAFETY.md` for the recommended
fix — it needs to go into each scheduled routine's stored prompt, which this session cannot edit.

_Original report filed automatically by the scheduled London-enrichment routine (group 4: June 26
/ March 25 / March 26 / May 25 / Sept 25). Resolution verified and filed by group 1
(April 25 / April 26 / Aug 25 / Dec 24 / May 26) at the repo owner's explicit request._
