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
- **`APIFY_INPUT`** (optional) — a JSON template for that actor's input, using **its own field names**
  (actor input schemas differ — there is no universal `search`/`location` contract). Put it in
  `~/.dear-hiring-manager/apify.json` (or point to a file). If it's absent, **fetch the actor's input
  schema first** (`GET https://api.apify.com/v2/acts/$APIFY_ACTOR` with the Bearer header) and build the
  input from the schema + the profile targets, then confirm with the user before running.

## Procedure
1. **Load the token without echoing it**, e.g.
   `APIFY_TOKEN=$(grep -m1 '^APIFY_TOKEN=' ~/.dear-hiring-manager/.env | cut -d= -f2-)` (fall back to the
   env var); same for `APIFY_ACTOR`. If the token is missing, stop and tell the user to put it in
   `~/.dear-hiring-manager/.env`. Never print the token or a command with it expanded.
2. **Build the search** from the profile: `Desired job title(s)` + `Target experience level` + preferred
   locations / remote. Only ask the user if those profile fields are blank.
3. **Build the actor input** from `APIFY_INPUT` (the actor's own field names) or, if unset, from the
   fetched input schema + profile targets (step above). **Run the actor** synchronously with **header
   auth, never the URL:**
   ```
   curl -sS -X POST \
     -H "Authorization: Bearer $APIFY_TOKEN" \
     -H "Content-Type: application/json" \
     -d "$APIFY_INPUT" \
     "https://api.apify.com/v2/acts/$APIFY_ACTOR/run-sync-get-dataset-items"
   ```
   No `?token=` in the URL; do not echo the token or the expanded command.
4. **Extract the application URL** per returned item. **Do NOT assume a field name** — actor outputs
   differ. Scan each item for URL-valued fields and pick the **direct company/ATS apply link** (a field
   like `applyUrl` / `externalApplyUrl` / `companyApplyUrl`, or any Greenhouse/Lever/Ashby/Workday URL),
   preferring it over the aggregator `jobUrl` (linkedin.com / indeed.com). Skip items with only an
   Easy-Apply / no external URL. If no item yields a usable URL, tell the user the actor's output shape
   doesn't expose one.

## Write urls.txt
- Prefer **direct ATS URLs** (Greenhouse / Lever / Ashby / Workday); note when only an aggregator URL exists.
- **De-dupe within this batch, and skip jobs already applied or done** — a URL in `applications.md` whose
  status is `submitted`, `rejected`, `interview`, `offer`, or **`filled`**. (`filled` = form filled and
  **parked awaiting the user's Submit** — a success, not a failure; re-queuing it would duplicate the
  application and add a second tracker row.) **Only `blocked` and `in-progress` re-queue** — a CAPTCHA /
  login block or a crashed run deserves a retry. `skipped` stays out unless the user lowers the fit
  threshold; re-queuing a `filled` job requires an explicit retry request from the user.
- Append to `~/.dear-hiring-manager/urls.txt` in the template format (a `# Company — Role` comment above
  each URL). Show the user the list + count; append or replace on their confirmation, never clobber silently.
- Tell them to run `/dear-hiring-manager:batch`.

## Rules
- Public postings only. The Apify token lives in `.env` / env — **never** in the repo, a URL, output, or a log.
- Authenticate with the `Authorization: Bearer` header, not a `?token=` query param.
- Prefer the direct application URL; an aggregator page (LinkedIn / Indeed) is a weak fallback that `apply`
  handles less well.
- Human-readable `urls.txt`; `/batch` needs nothing but that file.
