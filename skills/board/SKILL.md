---
name: board
description: Application tracker board. Read ~/.dear-hiring-manager/applications.md and render a status board (grouped by blocked/skipped/filled/submitted/rejected/interview/offer with counts and a response-rate summary), and update an application's status + date when the user reports progress (e.g. "mark Transcend as interview"). Use for /board or whenever the user wants to see or update their job-application pipeline.
---

# Tracker board

Read and maintain the application pipeline in `~/.dear-hiring-manager/applications.md` (the flat-file
tracker). Two jobs: **show** the board and **update** a status.

## Show
1. Read `applications.md`. If it's missing or has no rows, say so and point to
   `/dear-hiring-manager:apply` or `:batch`.
2. Render a board grouped by `status`, in pipeline order:
   **blocked → skipped → filled → submitted → rejected → interview → offer**.
   - A count per column header.
   - One line per application under its status: `company — role  (fit N · applied <date> · N flags)`.
   - A one-line summary: totals, and response rate =
     `(interview + offer) / (submitted + rejected + interview + offer)` — i.e. responses over everything
     ever submitted (status is single-valued, so an advanced app has left the `submitted` bucket). Show
     `n/a` when that denominator is 0.
3. Surface what's actionable:
   - `blocked` apps (CAPTCHA / unreadable form) → need your manual action; list them first.
   - `filled` apps not yet submitted → parked, awaiting the human's Submit.
   - `submitted` apps with no status change in >14 days → consider a follow-up.

## Update
When the user reports progress (e.g. "Transcend → interview", "Airbnb rejected"):
1. Find the matching row in `applications.md` by company/role.
2. Set its `status` and append a dated note in the flags/notes column (e.g. `interview 2026-07-20`).
   **Never delete history — append.**
3. Re-render the board.

## Rules
- Flat-file only; `applications.md` is the single source of truth, human-readable Markdown.
- Status lifecycle: in-progress → blocked | skipped | filled → submitted → rejected | interview | offer.
- Never fabricate a status — only reflect what the human reports or what `apply`/`batch` recorded.
