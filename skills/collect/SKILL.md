---
name: collect
description: Auto-collect live job-posting URLs into ~/.dear-hiring-manager/urls.txt for batch applying. Given role keywords (and optional location), search the web for current postings on supported ATS (Greenhouse/Lever/Ashby), drop stale/closed ones, de-dupe, and write a clean URL list. Use for /collect or whenever the user wants to gather jobs to apply to.
---

# Collect job URLs

Gather live posting URLs into `~/.dear-hiring-manager/urls.txt` so `/dear-hiring-manager:batch` can fill them.

## Input
- Role keywords + optional location/remote (e.g. "Customer Success Manager, US remote"). **If none are
  given, use the profile's `Desired job title(s)` + `Target experience level` + preferred locations as the
  search** (only ask the user if those are blank too).
- Optional count (default ~10).

## Procedure
1. **Search** the web for current postings matching the role/location, biased to supported ATS
   (`job-boards.greenhouse.io`, `jobs.lever.co`, `jobs.ashbyhq.com`) — the reliably fillable ones.
2. **Verify live**: open each candidate; a redirect to the board root / `?error=true` / no job title
   means stale → drop it. Keep only postings that load with a real title (indexed URLs go stale fast).
3. **Cheap pre-screen** against the profile/role so the list isn't full of obvious non-matches — this is
   a rough filter only; `batch` still runs the real fit-gate per URL.
4. **De-dupe** and write survivors (up to the requested count) to `~/.dear-hiring-manager/urls.txt`, one
   URL per line, each preceded by a `# company — role` comment for readability. Show the user the list
   first; append or replace on their confirmation — never clobber silently.
5. Tell the user how many were collected and to run `/dear-hiring-manager:batch`.

## Rules
- Public postings only — no logins, no scraping behind auth, no CAPTCHA solving.
- Human-readable `urls.txt`; flat-file, no extra API keys (uses web search).
- Verify-live before writing — a stale URL just wastes a batch tab.
- Honor the requested count; if fewer live ones were found, say so.
