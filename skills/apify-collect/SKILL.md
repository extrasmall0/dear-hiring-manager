---
name: apify-collect
description: Collect fresh job-posting URLs into ~/.dear-hiring-manager/urls.txt by running an Apify job scraper — the discovery source for /batch. Runs a configured Apify actor via the Apify API (Bearer-header auth) with a search built from the profile's desired titles + level + location, extracts the direct application URL per job, and de-dupes against already-submitted jobs. Use for /apify-collect.
---

# Apify collect

Gather fresh job URLs into `~/.dear-hiring-manager/urls.txt` (the single input to `/dear-hiring-manager:batch`)
by running an **Apify** job scraper — purpose-built, far fresher and more complete than web search.

## Config
- **`APIFY_TOKEN`** — your Apify API token. Read from `~/.dear-hiring-manager/.env` (`APIFY_TOKEN=...`) or
  the `APIFY_TOKEN` env var. **Never** put it in the repo, in a URL, in command output, or in any log
  (`.env` is gitignored).
- **`APIFY_ACTOR`** — the Apify job-scraper actor to run (a LinkedIn / Indeed / ATS-board actor of your
  choice). Set it in the same `.env`, or ask the user which actor to use.

## Procedure
1. **Load the token without echoing it**, e.g.
   `APIFY_TOKEN=$(grep -m1 '^APIFY_TOKEN=' ~/.dear-hiring-manager/.env | cut -d= -f2-)` (fall back to the
   env var); same for `APIFY_ACTOR`. If the token is missing, stop and tell the user to put it in
   `~/.dear-hiring-manager/.env`. Never print the token or a command with it expanded.
2. **Build the search** from the profile: `Desired job title(s)` + `Target experience level` + preferred
   locations / remote. Only ask the user if those profile fields are blank.
3. **Run the actor** synchronously and read its dataset items. **Authenticate with a header, never the URL:**
   ```
   curl -sS -X POST \
     -H "Authorization: Bearer $APIFY_TOKEN" \
     -H "Content-Type: application/json" \
     -d '<actor input JSON: search terms, location, maxItems ~25>' \
     "https://api.apify.com/v2/acts/$APIFY_ACTOR/run-sync-get-dataset-items"
   ```
   No `?token=` in the URL; do not echo the token or the expanded command.
4. **Extract the application URL** per item — prefer `applyUrl` / `externalApplyUrl` / `companyApplyUrl`
   (the direct company/ATS link) over the aggregator `jobUrl` (linkedin.com / indeed.com). Skip
   Easy-Apply-only items with no external URL.

## Write urls.txt
- Prefer **direct ATS URLs** (Greenhouse / Lever / Ashby / Workday); note when only an aggregator URL exists.
- **De-dupe within this batch, and skip only jobs already applied** — a URL in `applications.md` whose
  status is `submitted`, `rejected`, `interview`, or `offer`. **Re-queue** anything left `blocked`,
  `in-progress`, or `filled` — those were never actually submitted (a crash / CAPTCHA / parked form
  deserves another chance). `skipped` may re-queue too (it just re-hits the fit-gate, cheap).
- Append to `~/.dear-hiring-manager/urls.txt` in the template format (a `# Company — Role` comment above
  each URL). Show the user the list + count; append or replace on their confirmation, never clobber silently.
- Tell them to run `/dear-hiring-manager:batch`.

## Rules
- Public postings only. The Apify token lives in `.env` / env — **never** in the repo, a URL, output, or a log.
- Authenticate with the `Authorization: Bearer` header, not a `?token=` query param.
- Prefer the direct application URL; an aggregator page (LinkedIn / Indeed) is a weak fallback that `apply`
  handles less well.
- Human-readable `urls.txt`; `/batch` needs nothing but that file.
