---
name: cover-letter
description: Generate a tailored, truthful cover letter from the user's resume, profile, and a specific job description. Draft it in the user's voice, ground every claim in the resume (no fabrication), map it to the role, flag it for review, and save a copy. Used by the apply skill when a portal has a cover-letter field, or standalone via /cover-letter <job-url>.
---

# Cover letter

Draft a tailored cover letter for one role. Truthful, specific, concise — never a generic template.

## Inputs
- `~/.dear-hiring-manager/resume.*` and `profile.md` — the candidate.
- The job description + company. If given only a URL, open it and extract the JD first (see `apply`).

## Procedure
1. Read the resume, profile, and JD.
2. Draft the letter (~250–350 words, 3–4 short paragraphs):
   - **Open** with the role + company by name and a one-line hook tying the candidate to the company's
     mission / the JD's core problem.
   - **Body**: 2–3 concrete, resume-backed proof points mapped to the JD's top requirements — use the
     real metrics from the resume (e.g. "115% NRR", "grew a EUR 2–2.5M ARR book"). No claim that is not
     in the resume.
   - **Close**: brief, forward-looking, a call to connect.
3. Match the candidate's register from the resume; plain, confident, specific. No buzzword soup, no
   clichés ("I am writing to apply…", "team player", "hit the ground running").
4. **Flag it review-me** and save to
   `~/.dear-hiring-manager/runs/<company-role-slug>/cover-letter.md`. If a form field expects it, fill
   that field with the text.

## Rules
- **Truthful**: every claim traces to the resume/profile — no invented employers, titles, dates, or numbers.
- Tailored to THIS JD/company — if it could be pasted into another application unchanged, it's too generic.
- The human always reviews before Submit; the draft is a flagged starting point.
- Flat-file: the saved copy is human-readable Markdown.
