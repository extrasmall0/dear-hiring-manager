# dear-hiring-manager — Plan

> A 24/7 agent that writes 'Dear Hiring Manager' so you don't have to.

Human-in-the-loop job-application filler. AI does **understanding**; deterministic code +
the browser do **execution**. Ships as a Claude Code plugin (interactive), designed to grow
into an unattended daemon later.

## Locked decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Phase 1 runtime | **Claude Code plugin/skill** | Zero extra API key, runs on user's Pro/Max sub, start straight in the CLI |
| Brain | **Claude** (single provider) | Already in Claude Code; no GPT split |
| Fill mechanism (P1) | **Playwright MCP browser, stop before Submit** | Works on all ATS; visible tab = best human review; matches "one tab per app, stop at submit" |
| Storage | **Markdown/YAML flat files** | Human-readable, git-friendly, zero deps, Claude reads natively; `answers.md` doubles as lightweight RAG |
| Memory | **Append-only Q&A log**, no vector RAG in P1 | Few hundred rows; grep + LLM judgment. Add embeddings only >~500 rows or when match quality drops |
| Update strategy | **Never auto-overwrite** | Human edits → append new row w/ company/job tag + date; recent+specific ranks first |
| CAPTCHA / anti-bot | **Do not evade** | Hit CAPTCHA → degrade to human, stop. Evasion = rabbit hole + ToS risk |
| Deterministic HTTP submit | **Phase 3, not now** | Shines unattended; interactive assisted-apply wants a visible browser |

## Architecture principle

**AI = brain (understand page, fields, JD, matching, cover letter). Code + browser = hands/feet
(open, click, type, wait).** Never let the LLM be both brain and hands — that's the common mistake.

## Storage layout

Repo = the plugin (shipped). User data = home dir (private, git-ignored by living outside repo).

```
~/.dear-hiring-manager/
  profile.md            # identity, contact, work-auth, EEO/demographics, standard screening Qs (all the "select No")
  answers.md            # append-only Q&A memory (the lightweight RAG)
  applications.md       # tracker: one row per application (status, fit score, date, flags)
  resume.pdf|.md        # registered resume
  runs/<job-slug>/      # per-application artifacts (extracted JD, filled values, cover letter)
```

`profile.md` must be **complete**: legal name, contact, location, work authorization / visa /
sponsorship, willing-to-relocate, EEO gender/race/veteran/disability, no-compete = No,
illegal-activity = No, felony = No, references, links (LinkedIn/GitHub/portfolio), salary
expectation, notice period. Anything screening forms ask.

## Phases

### Phase 0 — Onboarding (dead simple)
Install plugin → `/onboard` runs an interview → writes `profile.md`, registers resume, seeds
`answers.md`. Re-runnable to edit.
- **Done when:** `~/.dear-hiring-manager/profile.md` fully populated via interview, resume
  registered, `answers.md` seeded. `/onboard` re-runs to edit any field.

### Phase 1 — Single job, assisted apply, interactive
Input one job URL →
1. Playwright MCP opens a browser tab, reads DOM, extracts JD + company → append `applications.md`.
2. Resume × JD fit score.
3. Fill every field from `profile.md` + `answers.md`.
4. Unseen open-ended question → fill best-guess **+ flag** (or stop and ask, per confidence).
5. **Stop before Submit.** Human reviews the visible tab, edits, clicks Submit.
6. After submit: agent asks "what did you change?" → append new/edited Q&A to `answers.md`.
- **Done when:** on 2–3 real postings across ≥2 ATS (Greenhouse/Lever/Ashby), agent opens
  browser, fills all mappable fields, flags unknowns, stops before submit, and appends the
  human's edits back to `answers.md`.

### Phase 2 — Batch + tracking + cover letter
- Batch import N URLs → loop P1, each own tab, all stop before submit, review together.
- Application tracker board (filled / submitted / rejected / interview).
- Cover-letter generation when a portal has the field. Low-fit auto-skip.

### Phase 3 — Unattended daemon + deterministic submit
- Agent SDK + API key (post-2026-06-15 billing reality: `-p`/SDK use own quota, always-on → API key).
- Greenhouse/Lever/Ashby via HTTP/GraphQL deterministic submit (no browser); Workday-type → browser fallback.
- Tiered auto-submit: **T1** clean form auto / **T2** CAPTCHA or low-confidence → fill & stop / **T3** skip.
- 24/7 email + job-board monitoring → discovery DB (introduce **SQLite** here for state/volume).
- launchd/cron scheduling.

**Migration:** `profile.md` / `answers.md` carry through untouched. SQLite added only at P3.

## Non-goals (guardrails)
- No anti-bot / CAPTCHA evasion.
- No auto-submit in P1/P2 — human owns the last inch.
- No vector RAG until the flat file demonstrably hurts.
- No multi-provider brain.
