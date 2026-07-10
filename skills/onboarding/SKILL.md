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

4. **Interview the gaps, section by section.** Fill every field in the template. Be efficient —
   batch related questions, accept short answers.
   - Identity, contact, links.
   - Location & relocation, remote/onsite preference.
   - **Work authorization**: country, "authorized to work?", "require sponsorship now or in future?",
     visa status. These appear on almost every form — get them exact.
   - Compensation expectation, notice period / earliest start.
   - **Standard screening (legal attestations)** — the template leaves these **blank**. Propose the
     safe defaults (non-compete: No, felony: No, illegal activity: No, accommodation: No, previously
     employed here: No, 18+: Yes) and have the user confirm each or override. Write a value **only
     after explicit confirmation**. If the user skips, leave it blank — blank means *unconfirmed*, and
     apply will flag it rather than attest.
   - **EEO / demographics** (gender, race/ethnicity, veteran, disability): state clearly these are
     **voluntary**; "prefer not to answer" is a valid, respected answer. Record exactly what they say.
   - References.

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
