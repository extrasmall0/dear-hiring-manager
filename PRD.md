# dear-hiring-manager — PRD (original design)

> Source design, captured verbatim-in-spirit. `PLAN.md` is the engineered/simplified version;
> this file preserves the original intent + a traceability map from every idea to where it lands.
> Reference project: https://github.com/torontodeveloper/job-application-agent

## Problem

Filling job-application forms is painful, repetitive, and standardized across most ATS. High-value,
low-glory. A human-in-the-loop agent can do 99% of the drudgery and leave the human the 1% decisions.

## Original flow (11 steps)

1. Provide personal info + resume **once** (or read from a known location). Must be complete:
   contact, demographics (gender/race), history, and the standard screening questions
   (non-compete, illegal activity, felony) — all default to **No**. Prepare the full set.
2. Tell it which job — a URL, or read from a DB/CSV. Minimize API keys (avoid Notion etc.), keep
   storage human-readable (SQL is powerful but not human-friendly).
3. AI opens the page, analyzes the whole application structure (DOM vs vision? Playwright vs browser agent?).
4. AI extracts job requirements + company info, records to DB.
5. Read resume + JD, score job↔candidate fit.
6. AI understands each field, pulls from RAG / info sheet, fills it.
7. **Self-improving memory:** for each question, check RAG for a similar prior Q&A. If unseen, either
   stop there, or fill-by-judgment + emit a **warning + flag** so the human knows this application hit
   a snag and can review the flagged spot. After the human fills/edits, AI does a whole-application
   review and auto-records into RAG — a lightweight RLHF loop. (Update strategy for similar questions
   must be thought through.)
8. (AI generates a tailored cover letter from the JD.)
9. Everything filled; page parked at the Submit button.
10. Human reviews — submit if good, edit if not.
11. Each application in its own tab/page; all left parked; human reviews them one by one at the end.

## Tech stack (original)

| Layer | Tool | Role |
|-------|------|------|
| Browser automation | Playwright | open, click, fill, handle page interactions |
| Brain | LLM (orig. GPT-5.4 → **we use Claude**) | understand page/JD, match info, write cover letter |

## Three design decisions (the valuable ones)

1. **Pick a specific, painful, narrow scenario** — filling application forms. Not a general agent.
   Painful, standardized, bounded error cost (human reviews), easy-to-measure value.
2. **Human-machine collaboration, not full automation.** 100% automation costs you 90% of scenarios.
   Accept 99% + human does the last 1% → 10× the reachable problem space, better UX (user keeps control).
3. **AI solves understanding; traditional code solves execution.** AI = brain (page structure, field
   meaning, JD, matching, cover letter). Playwright/code = hands/feet (open, click, type, wait, states).
   The common mistake is the reverse — clumsy, error-prone, hard to debug.

## Good ideas (backlog)

1. 24/7 email monitor → scrape new job links/JDs → discovery DB for downstream agents.
2. One tab per application, parked before Submit.
3. A dedicated job-application Gmail; grant the agent read access.
4. **Memory & learning** — remember prior answers, preferred phrasings, refused questions. RAG:
   after each edit, save the answer; next time query RAG first, then match to current JD/company.
   Similar-question update strategy must be designed.
5. **Tracking & management** — after applying: track status (viewed/rejected/interview) on a unified
   board. Turns an "apply tool" into a "job-search management system."
6. **Anti-anti-scraping** — many sites have bot detection/CAPTCHA. (We treat this as a **non-goal**.)
7. Job-fit scoring.
8. Auto-collect the URL list.
9. If a cover-letter portal is detected, generate a cover letter.
10. **Batch apply** — import 10 links, fill each, review all at the end.

## CLI-integration analysis (why Claude Code fits)

- career-ops already proves half of it: its **assisted-apply** mode drafts every answer from the CV
  and fills the real form, leaving only the Submit click to the human. Not auto-submitting is a
  product choice, not a technical ceiling.
- Real integration is **headless**: `claude -p` non-interactive, Agent SDK (Python/TS) for full
  programmatic control, `--allowedTools` / `--permission-mode` to pre-authorize, skills usable in
  `-p`. Daemon (launchd) does scheduling + state machine; each job spawns a headless session with
  Playwright MCP; `--output-format json` writes back to SQLite. CLI = brain, pipeline = body.
- **Two constraints (cost/benefit, not feasibility):**
  - **Billing.** career-ops' "free magic" rides the user's existing Pro/Max sub in interactive
    sessions. After **2026-06-15**, `claude -p` and Agent SDK use a separate quota pool; always-on
    shared automation → Anthropic recommends metered API keys. 24/7 daemon on a subscription doesn't
    hold; API has real cost. (Hence career-ops externalizes cost to the user's own CLI session.)
  - **CAPTCHA & permissions.** Unattended = swap "human approves each risky action" for an allowlist
    decided up front. Claude will not (and should not) solve CAPTCHAs. Hit one → degrade to human.
    Fits the tiered design: **T1** clean form auto / **T2** CAPTCHA or low-confidence → fill & wait for
    human Submit (== assisted apply) / **T3** skip.
- **Deeper angle: form-filling may not need an LLM-driven browser at all.** Greenhouse, Lever, Ashby
  public application forms are backed by plain HTTP requests (Ashby is a GraphQL mutation) — structured,
  directly constructable, an order of magnitude more stable than clicking DOM (minus the few with
  reCAPTCHA). Same path you already poll for discovery. LLM only writes the open-ended answer content;
  submission itself is deterministic code. Browser automation stays as the fallback for Workday-type
  portals.
- **So: two layers.** A distribution layer as a **skill** — runs interactively in any CLI like
  career-ops, default no auto-submit, safe & uncontroversial. A runtime layer as your own **daemon**
  using the Agent SDK for fully-automatic tiered submission. Same skill files, two run modes. README
  one-liner: *"runs interactively in your CLI, or unattended as a daemon."* That dual-mode is the
  cleanest differentiator vs career-ops.

## Traceability — every idea → where it lands

| PRD idea | Status / Phase |
|----------|----------------|
| Specific painful scenario | Core, built |
| Human-in-loop, park before Submit | Phase 1 — built |
| AI-understands / code-executes split | Architecture principle — built |
| Provide info + resume once | Phase 0 onboarding — built & tested |
| Complete profile (demographics, screening → No) | Phase 0 `profile.md` — built |
| Memory & learning (RAG) | Simplified to append-only `answers.md` — Phase 1 built |
| Similar-question update strategy | "Never overwrite, newest+specific wins" — decided |
| Extract JD + company → DB | Phase 1 apply skill → `applications.md` |
| Job-fit scoring | Phase 1 apply skill (not yet live-tested) |
| Understand field → fill from RAG/profile | Phase 1 apply skill |
| Warning + flag on unseen question | Phase 1 apply skill (flag rule) |
| Whole-app review → record to RAG (RLHF-lite) | Phase 1 "learn after submit" step |
| Cover letter from JD | Phase 1 (if field detected) / Phase 2 |
| Park at Submit, human decides | Phase 1 — built |
| One tab per application | Phase 1 (single) → Phase 2 (batch) |
| Tracking board (viewed/rejected/interview) | `applications.md` seeded; board Phase 2 |
| Batch apply (import N links) | Phase 2 |
| Auto-collect URL list | Phase 2 |
| 24/7 email monitor → discovery DB | Phase 3 |
| Dedicated Gmail, agent read access | Phase 3 |
| Deterministic HTTP/GraphQL submit (GH/Lever/Ashby) | Phase 3 |
| Dual-mode daemon (Agent SDK, tiered submit) | Phase 3 |
| Choose storage: human-readable, few API keys | Flat Markdown files — decided, built |
| Anti-anti-scraping / CAPTCHA evasion | **Non-goal** — degrade to human |

## Deliberate simplifications vs original

- **RAG → flat `answers.md`.** No vector DB / embeddings until >~500 rows or match quality drops.
  Honors the "few API keys + human-readable" constraint.
- **RLHF loop → append-only log.** Human edits appended as new entries; newest+specific wins. No
  auto-overwrite, no training loop.
- **Single provider (Claude), not GPT.** Already in Claude Code.
- **HTTP deterministic submit deferred to Phase 3.** Interactive assisted-apply wants a visible
  browser tab for human review; deterministic submit shines unattended.
