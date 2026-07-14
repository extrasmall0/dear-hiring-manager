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
   - Read it. Prefill name, contact, links, location, education, and an experience summary into the
     profile. Then only ask the user to confirm/correct those, not retype them.

4. **Interview the gaps via the option-picker.** For every field whose answer is a discrete choice,
   ASK WITH THE `AskUserQuestion` OPTION-PICKER (not prose) — present sensible options plus "prefer not
   to answer" where relevant; the user clicks (or types via "Other"). Batch up to 4 questions per
   picker call to minimize rounds. Use free-text prompts only for genuinely open fields (name, contact,
   links, an exact salary number, references). Fill every template field.
   - Identity, contact, links — free text (or prefilled from the resume; just confirm).
   - Location & relocation, remote/onsite — **options** (relocate: Yes/No; preference: Remote / Hybrid /
     Onsite / Open).
   - **Work authorization** — **options**: authorized to work there (Yes/No); require sponsorship now or
     in future (Yes / No / Not now, yes later); + country and visa status. On almost every form — get exact.
   - Compensation — **options** for salary (a few ranges + "market/negotiable", Other for an exact number)
     and notice period / earliest start (2 weeks / 1 month / immediately / flexible).
   - **Application preferences** — **minimum fit score to apply** via **options** (e.g. 40 / 50 / 60 / 70,
     default 50). Hard filter: apply scores each job first and STOPS before filling if below this number.
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

6. **Confirm done.** Print the profile path and a one-line completeness check (any field still blank).
   Tell them they can now run `/dear-hiring-manager:apply <job-url>`.

## Rules
- Human-readable Markdown only. No database, no API keys.
- Never fabricate legal/EEO answers. Defaults are *suggestions to confirm*, not silent choices.
- Store data under `~/.dear-hiring-manager/`, never in the plugin repo.
