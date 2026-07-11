# dear-hiring-manager
A 24/7 agent that writes 'Dear Hiring Manager' so you don't have to.

Assisted job-application filler for Claude Code. It extracts the job description, scores your fit,
fills every form field from your profile and answer memory, flags anything uncertain — and **stops
before Submit** so you own the last click. AI understands; the browser executes.

## Quick start

1. **Install the plugin** (local dev):
   ```
   /plugin marketplace add /Users/you/PycharmProjects/dear-hiring-manager
   /plugin install dear-hiring-manager@dhm-local
   ```
   Then **restart Claude Code** so the bundled Playwright MCP server (`@playwright/mcp`) loads;
   approve it when prompted. Plugin commands are namespaced as `/dear-hiring-manager:<command>`.

2. **Onboard once** — build your profile and register your resume:
   ```
   /dear-hiring-manager:onboard
   ```
   Writes `~/.dear-hiring-manager/profile.md` (identity, work authorization, EEO, screening answers).

3. **Apply to a job** — fill a posting, review, submit yourself:
   ```
   /dear-hiring-manager:apply https://boards.greenhouse.io/acme/jobs/123456
   ```
   Opens the posting in a browser, fills the form, parks at Submit. You review the flagged fields and
   click Submit. Your edits are learned back into `~/.dear-hiring-manager/answers.md`.

## Your data

Everything lives in `~/.dear-hiring-manager/` as plain Markdown — human-readable, git-friendly, no
database, no extra API keys:

- `profile.md` — who you are, work auth, EEO, standard screening answers
- `answers.md` — append-only Q&A memory (the lightweight "RAG")
- `applications.md` — tracker (one row per application)
- `resume.*` — your registered resume

## Roadmap

Phase 1 (interactive assisted apply) is what ships here. Later phases: batch apply, application
tracking, cover-letter generation, and an unattended daemon with deterministic HTTP submit.

## Guardrails

No auto-submit. No CAPTCHA evasion. No fabricated legal/EEO answers. Human owns the last inch.
