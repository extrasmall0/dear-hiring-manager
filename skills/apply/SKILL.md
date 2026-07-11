---
name: apply
description: Assisted job-application filler. Given a job posting URL, open it in a browser via Playwright, extract the JD and company, score resume-to-JD fit, fill every form field from the user's profile and answer memory, flag anything uncertain, and STOP before Submit for human review. After the human submits, capture their edits back into answer memory. Use for /apply or whenever the user wants help applying to a specific job.
---

# Apply (assisted, human owns Submit)

Fill one job application from a URL. **You never click Submit.** You fill, flag, and hand back a
browser tab parked at the Submit button for the human to review, edit, and send.

Data lives in `~/.dear-hiring-manager/`: `profile.md`, `answers.md`, `applications.md`, `resume.*`.

## Preconditions
- If `~/.dear-hiring-manager/profile.md` is missing, stop and tell the user to run
  `/dear-hiring-manager:onboard` first.

## Procedure

1. **Open the posting.** Use the Playwright MCP tools to navigate to the URL. Take a page snapshot
   (accessibility tree / DOM) — do not rely on vision unless the DOM is unusable. **If the URL
   redirects to the board root or an error page (e.g. `?error=true`, only a job list / "Back to jobs",
   no job title or application form), the posting is closed or expired — stop and tell the user; do not
   proceed.**

2. **Extract JD + company.** Pull role title, company, location, and the job description. Detect the
   ATS from the URL/DOM (greenhouse, lever, ashby, workday, other). Append a row to
   `applications.md` (create from `${CLAUDE_PLUGIN_ROOT}/templates/applications.template.md` if missing)
   with `status=in-progress` (fit is filled in the next step; status advances as the run proceeds):
   `date | company | role | url | ats | fit | status | flags`.

3. **Score fit — FIRST FILTER (hard gate, nothing filled before it passes).** Read `resume.*` and the
   JD, give a 0–100 fit score with a 2–3 line rationale (matched strengths, gaps), and record it in the
   tracker row. Read the user's **Minimum fit score to apply** from `profile.md` (default 50 if unset).
   **If the score is below that threshold — or there's a hard-eligibility blocker (the role's country
   needs work authorization the user lacks) — STOP here:** set the tracker row `status=skipped` (reason:
   low-fit or ineligible), report the score, the threshold, and why, and do not fill the form. The user
   can override by replying "apply anyway" or lowering their threshold.

4. **Map every field — fill or flag, never silently skip.** Every required field must end up either
   filled or explicitly flagged; a blank required field is acceptable only when it is flagged. For each:
   - **Profile-backed** (name, contact, salary): fill from `profile.md`.
   - **Work authorization (per role country)**: answer for the ROLE's country — do not paste the
     profile's raw yes/no. E.g. authorized in NL+EU but the role is in the US → "authorized: No",
     "sponsorship: Yes". Flag whenever the role country is outside the user's authorized regions.
   - **Legal attestations** (non-compete, felony, illegal activity, accommodation, previously-employed,
     age): fill **only** from a confirmed, non-blank `profile.md` value. If blank/unconfirmed, **flag and
     leave for the human** — never auto-attest a legal answer from a default.
   - **Voluntary EEO** (gender, race/ethnicity, veteran, disability): use `profile.md` if set; if
     blank, select "prefer not to answer" / "decline to self-identify". Never flag, never block on these.
   - **Open-ended / unseen** (e.g. "Why this company?", custom screening): search `answers.md` for a
     matching prior Q&A. Prefer the newest, most company/role-specific match.
       - Match found → adapt and fill.
       - No match → write a best-guess grounded in resume + profile, fill it, and **flag** it.
   - **File uploads (resume/CV, cover letter)**: Playwright MCP sandboxes file reads to the workspace
     root. Copy `~/.dear-hiring-manager/resume.*` into `.playwright-mcp/` (gitignored), upload from
     there, then delete the staged copy. Do NOT enable `--allow-unrestricted-file-access` — staging
     keeps the sandbox intact instead of exposing every file on the machine.
   - **Comboboxes**: many ATS (Greenhouse/Ashby) render react-select, not native `<select>`. You MUST
     first **click** the combobox to open its menu, THEN type the option text, THEN press Enter.
     Typing/fill without opening first silently fails — the field stays empty (`Select...`). Re-snapshot
     to confirm each value stuck; `browser_fill_form` with type `combobox` does not work on react-select.
   - Confidence rule: high confidence → fill quietly. Low confidence or anything legal/sensitive with
     no profile answer → fill best-guess **and flag**, or leave blank and flag. When in doubt, flag.

5. **Cover letter.** If the form has a cover-letter field, generate one tailored to the JD (draft it,
   fill it, flag as review-me). Otherwise skip.

6. **CAPTCHA / anti-bot.** If you hit a CAPTCHA or bot check, **do not attempt to solve or evade it.**
   Stop, leave the tab as-is, and tell the human to complete that step manually.

7. **Stop before Submit.** Leave the browser tab open, scrolled to / parked at the Submit button.
   Print a review summary:
   - Fit score.
   - Fields filled (grouped).
   - **⚑ Flagged fields** needing the human's eyes — list each with what you guessed and why.
   - **Required fields still blank/untouched** — list every one; the human must resolve these before Submit.
   Update the tracker row `status=filled`. Tell the human: review the tab, fix flagged items, then click
   Submit yourself.

8. **Learn after submit.** When the human says they've submitted (or edited), ask what they changed.
   Route each change to the right place so future applications benefit:
   - **Profile-level fact** (salary expectation, location/city, phone, links, work authorization — a
     field that lives in `profile.md`): update `profile.md` so *every* future application picks it up
     and stops re-flagging it, not just keyword lookups.
   - **Company/role-specific or open-ended** (e.g. "Why this company?", a custom essay): **append** a
     new entry to `answers.md` (never overwrite), `source: human-edit`, tagged with company + role + date.
   - **Legal attestation the human confirms**: write the confirmed value to `profile.md` (now confirmed)
     and seed a matching `answers.md` entry.
   Then update the tracker row `status=submitted`.

## Rules
- AI understands; the browser + deterministic steps execute. Don't improvise clicks beyond filling.
- Never Submit. Never solve CAPTCHAs. Never fabricate legal/EEO answers.
- Fill or flag every field; never leave a required field silently blank.
- Flag internal profile inconsistencies you spot (e.g. phone country code vs stated location).
- Append-only memory: human edits win by being newer, not by overwriting.
- One posting per run. Batch is Phase 2.

## ATS notes (from dogfooding)
- **Greenhouse** (`job-boards.greenhouse.io`, or iframe-embedded e.g. `careers.airbnb.com`): fields are
  in the DOM; screening/EEO are react-select comboboxes (click → type → Enter). Works well. A `reCAPTCHA`
  usually guards Submit — fine, the human submits. Verified: Airbnb, Overstory, Transcend.
- **Stale/closed postings**: redirect to the board root with `?error=true` and no form → stop (step 1).
- **Ashby** (`jobs.ashbyhq.com`): JS single-page app; a static fetch sees only the shell. Drive it with
  the browser (Playwright renders the JS); don't rely on page-source scraping.
- **Lever** (`jobs.lever.co`): bot-blocks static fetches (HTTP 403). Use the browser.
- **Workday** (`*.myworkday.com`): usually requires creating an account/login before the form, plus
  custom widgets. Treat as Tier 3 — skip or hand to the human unless already signed in.
