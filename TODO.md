# TODO — dear-hiring-manager

Working backlog, done one by one. Durable across sessions. Phase detail in `PLAN.md`; original design in `PRD.md`.

## Done
- [x] Scaffold plugin (Phase 0 onboarding + Phase 1 apply)
- [x] Onboarding live-tested (resume parse, work-auth, seed answers)
- [x] Codex P1: namespace commands (`/dear-hiring-manager:onboard|apply`)
- [x] Codex P1: never auto-attest unconfirmed legal answers
- [x] EEO blank → "prefer not to answer" default
- [x] `marketplace.json` (installable via `/plugin`)
- [x] Apply live-tested on a real Airbnb Greenhouse form (browser fill end-to-end)
- [x] Fix: stage resume into Playwright allowed root before upload
- [x] Fix: work-auth per-country reasoning (role country vs authorized regions)
- [x] Fix: fit-gate — stop/ask before filling on near-zero fit or hard-ineligibility
- [x] Fix: fill-or-flag every field; never silently skip; stop-summary lists untouched required fields
- [x] Fix: react-select combobox mechanics documented
- [x] Fix: flag profile self-inconsistency (phone vs location)
- [x] `.gitignore` `.playwright-mcp/` + root screenshots; clean staged artifacts

## Next — Phase 1 polish
- [ ] Real end-to-end run on a MATCHING role (CSM) — verify full fill of every required field
- [ ] Configure Playwright MCP allowed roots so resume uploads from `~/.dear-hiring-manager` without a copy (cleaner than staging)
- [ ] Test "learn after submit" loop (append human edits → `answers.md`)
- [ ] Test onboarding confirm-legal-screening flow end to end
- [ ] Handle multi-step / paginated application forms
- [ ] Per-ATS quirks: Greenhouse iframe, Ashby SPA, Lever (bot-block), Workday

## Phase 2 — batch + tracking + cover letters
- [ ] Batch apply: import N URLs, fill each in its own tab, review together
- [ ] Application tracker board (filled / submitted / rejected / interview / offer)
- [ ] Cover-letter generation when the portal has the field
- [ ] Fit-gating auto-skip for low-fit roles
- [ ] Auto-collect URL list

## Phase 3 — unattended daemon
- [ ] Agent SDK daemon, tiered submit (T1 auto / T2 stop / T3 skip)
- [ ] Deterministic HTTP/GraphQL submit (Greenhouse API first; Lever/Ashby via browser)
- [ ] 24/7 email monitor → discovery DB (introduce SQLite)
- [ ] Dedicated job-application Gmail, read access
- [ ] Scheduling (launchd/cron)

## Optimizations / tech debt
- [ ] `answers.md` → embeddings/RAG when >~500 rows or match quality drops
- [ ] Phone country-code handling on international forms
- [ ] Screenshot/artifact retention policy
- [ ] Cost note: interactive (subscription) vs daemon (API key, post-2026-06-15)
