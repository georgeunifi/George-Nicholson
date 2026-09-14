# First directors — Hertfordshire

First named individual director per company, taken from the Companies House
officer record (`/company/{number}/officers`) on 13 September 2026.

Selection rule: earliest-appointed **active** officer whose role includes
Director and who has a date of birth (which excludes corporate directors).

Kept in a separate folder rather than appended to `Enriched/` so the 14-column
Enriched format stays intact. Join on `company_number`.

Companies House is blocked from the Claude cloud sandbox by the organisation's
egress policy (HTTP 403 on CONNECT, both `find-and-update.` and `api.` hosts) —
an API key does not change that. These were retrieved by same-origin `fetch`
from a page already open on the Companies House site in the browser on George's
laptop, throttled to ~850ms between requests. 241 lookups, zero blocks.

| File | Rows |
|---|---|
| Aug 25.csv | 7 |
| Sept 25.csv | 3 |
| Oct 25.csv | 47 |
| Nov 25.csv | 62 |
| June 26.csv | 61 |
| July 26.csv | 61 |

Known data issue: company 16807944 (MFB OSTEOPATHY LIMITED) is filed at
Companies House as "MONICA, Blackburn", so forename and surname may be
reversed. Verify before using the name in correspondence.
