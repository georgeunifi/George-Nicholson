# London enrichment — paused until Monday

Repo owner (george@unifinancialsolutions.co.uk) has asked for the "London company
enrichment" scheduled routine, and any related London/Herts enrichment automation,
to **stop running until Monday 2026-10-05**.

Filed Tue 2026-09-29.

## Instruction for any automated session picking up work here

If you are a scheduled/automated session (any London enrichment group, trigger
`trig_01FHdmVpL7KDY8xqqGMBzSoe` or similar) and you see this file present on `main`:

- Do **not** run any WebSearch enrichment, edit any `London/Enriched/*.csv` file,
  or push commits for this job.
- Do not treat this as a task to "complete" — just stop and exit without claiming
  work, same as if all files were finished.
- This file should be deleted (by the repo owner, or by an automated run explicitly
  resuming the job after Monday) once work is allowed to continue.

This does not affect unrelated work in this repo.
