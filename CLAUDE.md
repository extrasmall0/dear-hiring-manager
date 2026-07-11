# CLAUDE.md — dear-hiring-manager

Guidance for any agent working in this repo. Read this + `PLAN.md`, `PRD.md`, `TODO.md` first.

## What this is
A **Claude Code plugin** that assists job applications: extract the JD, score fit, fill the form via
Playwright, and **stop before Submit** for the human. Not a standalone agent — Claude Code is the
runtime; this repo is the skill/knowledge layer. (Phase 3 later adds a real daemon; see PLAN.md.)

## Architecture principle
**AI understands; code + the browser execute.** The LLM reads pages, fields, JD, matching, cover
letters. Playwright does open/click/type/wait. Never make the LLM the hands.

## Layout
- `.claude-plugin/plugin.json` · `marketplace.json` — plugin + local marketplace (`dhm-local`)
- `.mcp.json` — bundles Playwright MCP (`@playwright/mcp`)
- `commands/` — thin entry points (`/dear-hiring-manager:onboard`, `:apply`)
- `skills/onboarding/SKILL.md` — interview → `profile.md`, parse resume, seed `answers.md`
- `skills/apply/SKILL.md` — the assisted-apply procedure (the core logic)
- `templates/` — `profile` / `answers` / `applications` templates (copied to user data dir)
- `PLAN.md` (phases) · `PRD.md` (original design + traceability) · `TODO.md` (backlog, gitignored/local)

## User data (never in repo)
`~/.dear-hiring-manager/`: `profile.md`, `answers.md` (append-only Q&A "RAG"), `applications.md`
(tracker), `resume.*`. Flat Markdown — human-readable, no DB, no extra API keys.

## Invariants (do not break)
- **Never click Submit.** Human owns the last click.
- **Never solve/evade CAPTCHA.** Hit one → stop, hand to human.
- **Never auto-attest unconfirmed legal answers** (non-compete/felony/etc.). Blank in profile → flag,
  don't fill. Legal defaults are written only after explicit onboarding confirmation.
- **Voluntary EEO blank → "prefer not to answer"** (never flag, never block).
- **Work authorization is per role country** — answer for the role's country, not the profile's raw
  yes/no (NL+EU auth + US role → authorized No, sponsorship Yes).
- **Fit score is the FIRST FILTER** — below the profile's `Minimum fit score to apply` (default 50) or
  hard-ineligible → stop before filling anything.
- **Fill or flag every required field** — never leave one silently blank.
- **answers.md is append-only** — human edits win by being newer, never overwrite.
- Tracker status lifecycle: `in-progress` (JD extracted) → `skipped` (gated) / `filled` (parked at
  Submit) → `submitted`.

## Conventions
- Skills reference bundled files via `${CLAUDE_PLUGIN_ROOT}/...`.
- Plugin commands are namespaced `/dear-hiring-manager:<command>`.
- Playwright MCP sandboxes file reads to the workspace root — stage `~/.dear-hiring-manager/resume.*`
  into `.playwright-mcp/` (gitignored) before upload, then delete the staged copy. Keep the sandbox;
  do not use `--allow-unrestricted-file-access`.
- **After each tested increment: commit AND `git push`** on the working branch (Conventional
  Commits). Work on a branch, not `main` directly; merge to `main` when Phase 1 is complete.

## How to test (dogfood, don't just check structure)
1. `/plugin marketplace add <repo>` → `/plugin install dear-hiring-manager@dhm-local` → restart
2. `/dear-hiring-manager:onboard` (profile exists → offers edit)
3. `/dear-hiring-manager:apply <real-job-url>` → drive Playwright, verify fill/flag/gate, stop before Submit
