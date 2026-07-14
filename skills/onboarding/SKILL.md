---
name: onboarding
description: Interview the user and build their job-application profile — identity, contact, work authorization, EEO/demographics, and the standard screening answers (non-compete, felony, sponsorship) — plus register their resume. Use for /onboard or whenever the user wants to set up or edit their dear-hiring-manager profile before applying to jobs.
---

# Onboarding

Goal: produce a complete, human-readable `~/.dear-hiring-manager/profile.md` and register a resume,
so later applications fill themselves. Keep it fast — parse what you can, only ask for gaps.

## Procedure

1. **Ensure the data dir exists.** Create `~/.dear-hiring-manager/` if missing.

2. **Check for an existing profile.**
   - If `~/.dear-hiring-manager/profile.md` exists: show a short summary, ask whether to *edit a
     section* or *view*. Only touch what they name. Do not restart the interview.
   - If not: copy `${CLAUDE_PLUGIN_ROOT}/templates/profile.template.md` to
     `~/.dear-hiring-manager/profile.md` and continue.

3. **Register + parse the resume (do this first — it prefills most fields).**
   - Ask for the resume path. Copy it to `~/.dear-hiring-manager/resume.<ext>`.
   - Read it. Prefill name, email/phone, location, education, and an experience summary into the
     profile. Then only ask the user to confirm/correct those, not retype them. **Do NOT prefill
     LinkedIn/GitHub/portfolio URLs from the resume — ask for those directly (see the interview step).**

4. **Interview the gaps via the option-picker.** For every field whose answer is a discrete choice,
   ASK WITH THE `AskUserQuestion` OPTION-PICKER (not prose) — present sensible options plus "prefer not
   to answer" where relevant; the user clicks (or types via "Other"). Batch up to 4 questions per
   picker call to minimize rounds. Use free-text prompts only for genuinely open fields (name, contact,
   links, an exact salary number, references). Fill every template field.
   - Identity + email/phone/name — prefill from the resume, then confirm.
   - **Links (LinkedIn, GitHub, portfolio/website): ALWAYS ASK the user directly — do NOT read them from
     the resume, and NEVER guess a URL from the name.** In a PDF these are hyperlinks whose real target
     URL usually can't be recovered by text parsing, so any parsed or name-guessed value is likely wrong.
   - Location & relocation, remote/onsite — **options** (relocate: Yes/No; preference: Remote / Hybrid /
     Onsite / Open). Plus **home address** (street, city, state, ZIP) — free text; many forms require the
     full address, not just the city.
   - **Work authorization** — **options**: authorized to work there (Yes/No); require sponsorship now or
     in future (Yes / No / Not now, yes later); + country and visa status. On almost every form — get exact.
   - Compensation — **options** for salary (a few ranges + "market/negotiable", Other for an exact number)
     and notice period / earliest start (2 weeks / 1 month / immediately / flexible).
   - **Job-search targets** — **desired job title(s)** (free text), **target experience level** (options:
     Intern / Entry / Mid / Senior / Staff / Principal / Director), **years of experience** (options:
     0–2 / 3–5 / 5–8 / 8–12 / 12+, or an exact number — forms ask this constantly), and **minimum fit
     score to apply** (options 40 / 50 / 60 / 70, default 50; the hard filter apply scores each job
     against). Title + level also feed `apify-collect` as the default search.
   - **Standard screening (legal attestations)** — template leaves these **blank**. Offer the safe
     defaults (non-compete: No, felony: No, illegal activity: No, accommodation: No, previously employed
     here: No, 18+: Yes) as **options** and have the user confirm each or override. Write a value **only
     after explicit confirmation**. Skipped → stays blank (unconfirmed); apply flags it, never auto-attests.
   - **EEO / demographics** (gender, race/ethnicity, veteran, disability) — **options**, each including
     "prefer not to answer". State clearly these are **voluntary**. Record exactly what they pick.
   - Pronouns — **options** (He/him / She/her / They/them / Prefer not to say). References — free text
     or "available on request".

5. **Seed the answer memory.** If `~/.dear-hiring-manager/answers.md` is missing, copy
   `${CLAUDE_PLUGIN_ROOT}/templates/answers.template.md` there. Add a few starter entries from
   **confirmed, non-blank** profile fields (work authorization, sponsorship, relocation, salary) in the
   entry format, `source: profile`. Never seed unconfirmed or blank values (especially legal attestations).

6. **Seed the URL list.** If `~/.dear-hiring-manager/urls.txt` is missing, copy
   `${CLAUDE_PLUGIN_ROOT}/templates/urls.template.txt` there — this is the single input for
   `/dear-hiring-manager:batch`. The user fills it however they like (by hand, `/apify-collect`, or an Apify
   export); batch never needs anything but this file.

7. **Seed the secrets stub (optional Apify).** If `~/.dear-hiring-manager/.env` is missing, copy
   `${CLAUDE_PLUGIN_ROOT}/templates/env.template` there **empty**. **Never ask for the Apify token in the
   chat** (it would leak into the transcript) — just tell the user to open `~/.dear-hiring-manager/.env`
   in an editor and paste their `APIFY_TOKEN` + `APIFY_ACTOR` there if they want `/apify-collect`. It's
   optional; skip mention entirely if they don't use Apify.

8. **Confirm done.** Print the profile path and a one-line completeness check (any field still blank).
   Tell them they can now run `/dear-hiring-manager:apply <job-url>`, or paste URLs into `urls.txt` and
   run `/dear-hiring-manager:batch`.

## Rules
- Human-readable Markdown only. No database, no API keys.
- Never fabricate legal/EEO answers. Defaults are *suggestions to confirm*, not silent choices.
- Store data under `~/.dear-hiring-manager/`, never in the plugin repo.
