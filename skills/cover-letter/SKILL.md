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
1. Read the resume, profile, and JD. Find a hook: the JD's core problem, a recent company launch/news, or
   a concrete result you can speak to.
2. Draft the letter (**~300–400 words**, 3–4 short paragraphs; under 250 reads thin, over 500 won't get finished):
   - **Open with a specific hook — NOT "I'm excited/passionate to apply."** Tie the first line to company
     research / the JD's core problem / a concrete result. Specificity earns the second sentence read.
   - **Body**: each paragraph maps one JD requirement to a real, resume-backed story **with numbers**
     (2–3 measurable accomplishments, not a skills list). Mirror the JD's own language — "you need X, I did
     Y" signals a careful read. No claim that isn't in the resume.
   - **Close** with a direct call to action — ask for a conversation.
3. **Voice + tone**: match the candidate's register from the resume — professional but distinct, not a
   corporate template. Role-fit: engineering → substance over polish; sales → lead with results/numbers;
   creative → a distinct perspective.
4. **Humanize.** Run the draft through the **`humanize`** skill — no em/en-dashes, no parenthetical
   asides, no AI tells, vary sentence length, sound like the candidate. Plus: no generic skill claims
   without evidence, no over-explaining gaps or pivots, no ATS-breaking formatting (keep it plain text).
5. **Flag it review-me** and save to `~/.dear-hiring-manager/runs/<company-role-slug>/cover-letter.md`,
   then deliver it to the form (see below).

## Deliver it to the form
- **Textarea / "enter manually"** → fill the field with the letter text.
- **File upload ("Attach")** → write the letter to a real file first
  (`~/.dear-hiring-manager/runs/<slug>/cover-letter.txt`, or `.pdf`), stage it into `.playwright-mcp/`
  (Playwright's sandbox root), upload from there, then delete the staged copy. Accepted types are usually
  `pdf, doc, docx, txt, rtf`.

## Rules
- **Truthful**: every claim traces to the resume/profile — no invented employers, titles, dates, or numbers.
- Tailored to THIS JD/company — if it could be pasted into another application unchanged, it's too generic.
- The human always reviews before Submit; the draft is a flagged starting point.
- Flat-file: the saved copy is human-readable Markdown.
