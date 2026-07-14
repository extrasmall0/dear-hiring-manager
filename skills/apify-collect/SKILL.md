---
name: apify-collect
description: Collect fresh job-posting URLs into ~/.dear-hiring-manager/urls.txt using Apify job scrapers — the discovery source for /batch. Runs a configured Apify actor via the Apify API with a search built from the profile's desired titles + level + location, or ingests an Apify dataset export (CSV/JSON) you already ran. Extracts the direct application URL per job, de-dupes against already-applied. Use for /apify-collect.
---

# Apify collect

Gather fresh job URLs into `~/.dear-hiring-manager/urls.txt` (the single input to `/dear-hiring-manager:batch`)
via **Apify** — purpose-built job scrapers, far fresher and more complete than web search.

## Config
- **`APIFY_TOKEN`** — your Apify API token. Read it from the `APIFY_TOKEN` env var, or from
  `~/.dear-hiring-manager/apify.json`:
  `{ "token": "...", "actor": "<actor-id>", "input": { ... } }`. **Never** put the token in the repo or
  any tracked file.
- **Actor** — the Apify job-scraper actor to run (a LinkedIn / Indeed / ATS-board actor of your choice).
  Set it in `apify.json` (`actor`) or pass one. If none is configured, ask the user which actor to use.

## Path A — run the actor (automated)
1. Build the search from the profile: `Desired job title(s)` + `Target experience level` + preferred
   locations / remote. Only ask the user if those profile fields are blank.
2. Run the actor synchronously and read its dataset items:
   `POST https://api.apify.com/v2/acts/<actor>/run-sync-get-dataset-items?token=$APIFY_TOKEN`
   with the actor's input JSON (search terms, location, and a sane max like ~25 results).
3. From each returned item, extract the **application URL** — prefer `applyUrl` / `externalApplyUrl` /
   `companyApplyUrl` (the direct company/ATS link) over the aggregator `jobUrl` (linkedin.com / indeed.com).
   Skip Easy-Apply-only items that have no external URL.

## Path B — ingest an export (manual, no token)
- Given a path to an Apify dataset export you downloaded (CSV or JSON), parse it and extract the same
  application-URL field per row. Same URL preference as above.

## Write urls.txt
- Prefer **direct ATS URLs** (Greenhouse / Lever / Ashby / Workday); note when only an aggregator URL exists.
- **De-dupe**, and **skip any URL already in `applications.md`** — never queue an already-applied job.
- Append to `~/.dear-hiring-manager/urls.txt` in the template format (a `# Company — Role` comment above
  each URL). Show the user the list + count; append or replace on their confirmation, never clobber silently.
- Tell them to run `/dear-hiring-manager:batch`.

## Rules
- Public postings only. The Apify token lives in the env or `apify.json`, never in the repo or a tracked file.
- Prefer the direct application URL; an aggregator page (LinkedIn / Indeed) is a weak fallback that `apply`
  handles less well.
- Human-readable `urls.txt`; `/batch` needs nothing but that file.
