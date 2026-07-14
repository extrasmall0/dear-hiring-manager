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
4. **Sound human, not AI.** Write like a real person saying why they fit. Vary sentence length, use
   contractions, be direct. **Never use em-dashes or en-dashes (— –) in the letter, and never use
   parentheses for an aside or explanation.** Rewrite any such aside as a plain sentence or a comma clause.
5. **Avoid the AI tells + clichés**: "I'm passionate/excited…", "team player", "hit the ground running",
   "it's not just X, it's Y"; the words leverage, delve, robust, seamless, furthermore, moreover, deeply,
   intricate; forced rule-of-three lists; generic skill claims with no evidence; over-explaining gaps or
   pivots; fancy formatting that breaks ATS parsers. Keep it plain text.
6. **Flag it review-me** and save to `~/.dear-hiring-manager/runs/<company-role-slug>/cover-letter.md`,
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
