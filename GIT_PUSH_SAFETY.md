# Safe push protocol for scheduled enrichment routines

Filed 2026-09-30 after a force-push race destroyed (then coincidentally got superseded by redone
work — see `DATA_LOSS_WARNING.md`) progress on this job. Multiple scheduled sessions run
concurrently against this repo and each currently does an unconditional
`git push origin HEAD:main`, sometimes from a shallow clone. That is racy by construction: two
sessions pushing around the same time can silently replace each other's history instead of
getting a normal non-fast-forward rejection.

**This file cannot fix the problem by itself.** The unsafe push instruction lives in each
scheduled routine's own stored prompt (account-level scheduling config, outside this repo), which
no session working inside this repo can read or edit. The repo owner needs to update each
routine's prompt to use one of the patterns below instead of a bare `git push origin HEAD:main`.

## Recommended replacement (safest, minimal change)

Before every push, do a full (non-shallow) fetch and only push if the local branch is still a
fast-forward of `origin/main`; if not, merge and retry rather than forcing:

```bash
git fetch --unshallow origin 2>/dev/null || git fetch origin main
git merge origin/main --no-edit   # or: git rebase origin/main
git push origin HEAD:main
```

If the merge/rebase hits a real conflict (two sessions edited the same file's same rows), that
should stop the session and surface the conflict rather than silently forcing either side's
history over the other.

## Alternative approaches (pick one, don't mix)

- **Single coordinator**: one session merges all groups' results into `main`; other sessions push
  to their own branch/PR instead of `main` directly.
- **Per-group file namespace**: each group already owns a disjoint file list (per the routine's
  own instructions) — if each group pushed to its own branch and a separate step merged branches
  centrally, two sessions could never race on the same push.

Any of these prevents the "two unrelated histories, one silently replaces the other" failure mode
seen on 2026-09-29. A bare `git push origin HEAD:main` retry loop does not — it just repeats the
race.
