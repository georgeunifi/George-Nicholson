# Data loss warning — London enrichment job (2026-09-29)

A concurrent-session git race has destroyed completed work on `main`. Please read before running more enrichment routines.

## What happened

On this run, `origin/main` and a commit still held locally in this container from an earlier
run of the same job (`6865996...`, "Enrich Oct 25.csv: 40 more London companies") turned out to
be **two completely unrelated git histories with no common ancestor** (`git merge-base` returns
nothing). Somewhere in the chain of parallel scheduled sessions, one session's `git push
origin HEAD:main` overwrote another session's unmerged commits outright — not a normal
non-fast-forward rejection, an actual history replacement (consistent with shallow clones being
pushed as if they were the full history).

## Confirmed losses

These `London/Enriched/*.csv` files existed with real progress in the orphaned commit but are
**entirely absent from current `main`**:

| File | Rows done (orphaned commit) | Source size |
|---|---|---|
| May 25.csv | 5000 / 5000 (fully complete) | 5000 |
| Sept 25.csv | 2735 / 5000 | 5000 |
| Oct 25.csv | 3873 / 5000 | 5000 |
| Nov 25.csv | 4451 / 5000 | 5000 |

That's roughly 16,000 already-completed WebSearch-verified company lookups no longer reachable
from `main`.

## What I tried

I attempted to restore these four files from the orphaned commit into current `main` (a purely
additive `git show <sha>:<path> > <path>` restore — no existing file would have been touched,
since none of the four exist on current `main`). This was **blocked by the environment's
permission classifier** as "Irreversible Local Destruction" and it instructed me to stop and let
a human decide, rather than try another tool to achieve the same restore.

The orphaned commit (`6865996e9ad7e9a4997db311a4fd67b7a7e9012a`, and its full history down to
root commit `7e93fdd`) was still present in this container's local git object store as of this
run. It may still be recoverable by:

```
git fetch origin
git show 6865996e9ad7e9a4997db311a4fd67b7a7e9012a --stat   # confirm it's still there
git show 6865996e9ad7e9a4997db311a4fd67b7a7e9012a:"London/Enriched/May 25.csv"   > "London/Enriched/May 25.csv"
git show 6865996e9ad7e9a4997db311a4fd67b7a7e9012a:"London/Enriched/Sept 25.csv" > "London/Enriched/Sept 25.csv"
git show 6865996e9ad7e9a4997db311a4fd67b7a7e9012a:"London/Enriched/Oct 25.csv"  > "London/Enriched/Oct 25.csv"
git show 6865996e9ad7e9a4997db311a4fd67b7a7e9012a:"London/Enriched/Nov 25.csv" > "London/Enriched/Nov 25.csv"
git add "London/Enriched/May 25.csv" "London/Enriched/Sept 25.csv" "London/Enriched/Oct 25.csv" "London/Enriched/Nov 25.csv"
git commit -m "Restore May/Sept/Oct/Nov 25 enrichment lost in force-push race"
git push origin HEAD:main
```

**This is only possible from a container/session that still has that commit object.** If no
running or future session has it cached locally and it isn't reachable from any ref on the
remote, it will eventually be garbage-collected and the work will be permanently lost. Act
promptly.

## Why I didn't just redo the work

Restarting May 25.csv / Sept 25.csv / Oct 25.csv / Nov 25.csv from scratch would burn WebSearch
budget re-doing lookups that already exist, and would create duplicate/conflicting content once
(if) the orphaned data is recovered. I left those four files alone and worked on `March 26.csv`
instead (safe: no conflicting history, straightforward continuation).

## Root cause worth fixing

Multiple independent scheduled sessions are each doing `git push origin HEAD:main` against what
may be shallow/independent clones. That pattern is racy by construction and appears to have
caused this exact kind of silent, total data loss more than once already (this container had
already had one earlier push overwritten before this run started). Recommend before running more
of these routines:
- Have a single writer/coordinator merge results instead of N sessions pushing straight to `main`, or
- Ensure every session does a full (non-shallow) `git fetch` + fast-forward-only `pull` immediately
  before every push and refuses to push on divergence rather than forcing, or
- Move each session's output to its own branch/file namespace and merge centrally.

_Filed automatically by the scheduled London-enrichment routine (group 4: June 26 / March 25 /
March 26 / May 25 / Sept 25). `PushNotification` also appears broken in this session (rejects its
own required `status` field), so this file is the only place this warning could be surfaced._
