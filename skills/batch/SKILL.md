---
name: batch
description: Batch job-application filler. Given several job URLs (or a urls.txt file), run the assisted-apply flow for each in its own browser tab — fit-gate, fill-or-flag, stop before Submit — skipping stale or low-fit postings, then present one combined review table so the human can approve each tab one by one. Use for /batch or whenever the user wants to apply to multiple jobs at once.
---

# Batch apply (many jobs, one review)

Fill N applications, each in its own tab, all parked before Submit. The human reviews the whole batch at
the end. This **orchestrates the `apply` skill once per URL** — identical invariants, fill-or-flag, and
fit-gate. You never Submit anything.

## Input
- URLs passed as arguments (space- or newline-separated), OR
- if none are given, read `~/.dear-hiring-manager/urls.txt` (one URL per line; skip blank lines and
  lines starting with `#`).

## Preconditions
- `~/.dear-hiring-manager/profile.md` must exist — else stop and tell the user to run
  `/dear-hiring-manager:onboard` first.

## Procedure

1. **Collect the URL list** (args or `urls.txt`), de-dupe, and report the count. If more than ~10 URLs,
   process the first 10 this run and tell the user the rest were deferred (avoid a runaway).

2. **For each URL, in its own tab:**
   - Open a **new** browser tab for it (`browser_tabs` action `new`) — one application per tab, all kept open.
   - Run the `apply` skill procedure on that tab:
     - Stale/closed posting (redirect to board root / `?error=true`, no form) → mark `skipped`, move on.
     - Extract JD + company; score fit; **fit-gate**: below the profile's `Minimum fit score to apply`
       or a hard-eligibility blocker → mark `skipped`, do **not** fill, move on.
     - Otherwise fill-or-flag every field; generate a cover letter if the portal has one; **stop before Submit**.
   - Update the row in `applications.md` (`status`: skipped | filled | blocked; record fit + flag count).
   - On any hard error (CAPTCHA, unreadable form) → set that row's `status=blocked` with the reason,
     leave the tab open, and continue; **never let one URL abort the batch**. Never leave a row `in-progress`.

3. **Combined review (brief).** After all URLs, print ONE table: `company · role · fit · status · #flags · tab`.
   **List `blocked` apps (CAPTCHA/unreadable) FIRST, in red (🔴) — they need your manual action.** Then
   under each filled app, one 🔴 line per flagged/blank field (🔴 = the red; terminal has no text color).
   No dump of filled fields. Tell the human: handle the 🔴 tabs/items, then Submit each yourself.

4. **Never Submit.** Every filled tab stays parked at its Submit button for the human.

## Rules
- One tab per application; leave them all open for the human's final pass.
- Same invariants as `apply`: never Submit, never solve CAPTCHAs, never auto-attest unconfirmed legal
  answers, EEO blank → prefer-not-to-answer, react-select combobox = click-to-open → type → Enter.
- A failure on one URL is recorded and skipped — it never stops the batch.
- One batch per run; keep it to ~10 URLs.
