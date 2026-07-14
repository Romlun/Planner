# PROJECT_STATE.md — UA House CHW Program Planner

**This file is written and maintained by Claude only.** The Code Agent (Codex)
edits application code on Claude's instruction but never edits this file.
If you're a Claude session picking this up cold, read this whole file before
touching anything, then confirm your understanding back to the operator
before acting.

## What this is
A single-user planner/tracker for UA House's Anthem CalAIM Community Health
Worker (CHW) program build-out — readiness through launch. Originally
generated from `UA_House_CHW_Program_Tracker.xlsx` (114 tasks across 7 phases
+ an "Ongoing / Cross-Phase" bucket), now a live app backed by a real
database instead of a static spreadsheet.

## Owner / access model
- Single user (Roman). Explicitly chosen: no login inside the app, open
  read/write via Supabase anon key.
- Vercel Deployment Protection (SSO) is ON at the platform level — this is
  intentional now that it's single-user, not a bug to fix. It means the app
  itself has no auth, but Vercel puts its own login in front of the whole
  site. Don't "fix" this without checking with the operator first.

## Stack
- **Frontend**: single static `index.html` — no framework, no build step.
  Vanilla JS, CSS custom properties for theming. Fonts via Google Fonts CDN
  (Fraunces / Inter / IBM Plex Mono). Supabase JS client via jsdelivr CDN.
- **Backend**: Supabase Postgres.
  - Project name: `Planner`, ref/id: `erejrnomrpxomdqubhkx`, region ca-central-1
  - Table: `public.tasks` (id, phase, track, pacing, category, task, owner,
    target_date, status, source, notes, sort_order)
  - `id` auto-assigns via `tasks_id_seq` (ALTER'd onto the existing integer
    PK) — next free id is 114+. Don't hardcode ids in future seed data.
  - RLS: anon can SELECT, UPDATE, and INSERT on `tasks`. No DELETE policy —
    the task list can be added to but not destroyed via the public client.
  - Anon/publishable key is embedded in `index.html` client-side — expected
    and safe *as long as* RLS stays scoped this way. If RLS policies change,
    re-check that this exposure is still appropriate.
- **Hosting**: Vercel project `planner` (id `prj_ObmYZAPmjamejPxvnLRdgU3Ed6nn`),
  team "Roman's projects" (`team_onqVX5EAyfn5vbEvmLlmlwqc`). Connected to
  GitHub, auto-deploys to production on every push to `main`.
  Domains: `planner-delta-six.vercel.app`, `planner-ecm-os.vercel.app`.
- **Repo**: `https://github.com/Romlun/Planner`, local clone at
  `/Users/romanlunickin/Documents/Planner` on Roman's laptop (macOS,
  Desktop Commander-connected). Single `main` branch, direct pushes, no PRs.

## Standing decisions
- No login inside the app; RLS is the only real access control, and it's
  intentionally permissive (see Owner/access model above).
- Direct git push is the workflow — no PR review, since it's a single
  developer. Claude pushes directly when the operator says so, with a
  heads-up before pushing since it's connected to an auto-deploy pipeline.
- Claude is the sole writer of this file. Codex/the Code Agent edits
  `index.html` (and any future app files) but never this file.
- Reference content in the Reference tab that isn't from the source
  spreadsheet is explicitly labeled "(added)" in the UI so it's clear what's
  sourced vs. synthesized.

## Current feature set (as of commit 6960336)
- **Dashboard** (default tab): stat tiles (Done/In Progress/Overdue/
  Remaining), This Week / Coming Up (grouped by week) / Overdue / No Fixed
  Date sections. Every task row here is fully interactive.
- **Timeline**: gate-rail view of the 7 phases with real dates, objectives,
  exit gates, and per-phase progress; click a phase to jump to its filtered
  task list.
- **Tasks**: full list, search + filter by phase/track/status, editable
  status/owner per row (persists to Supabase), "+ Add Task" modal (phase,
  track, category, target date, owner, status, notes — inserts live).
- **Reference**: Key Recommendations, Regulatory & Rate Reference, Anthem
  CHW Contract Terms — expanded beyond the source spreadsheet with clearly
  labeled additions.
- **Removed**: the Financial Model tab (removed per operator request in
  commit 6960336) — do not re-add without being asked.

## Design tokens (keep consistent in future edits)
- Palette: paper `#F2F4F3`, ink `#152521`, teal `#1F6F5C` / teal-dark
  `#134A3D` (primary accent), amber `#C77D2E` (Anthem track / warnings),
  red `#B23A32` (blocked/overdue), slate `#7C8A87` (muted/secondary).
- Type: Fraunces (headers/serif), Inter (body/UI), IBM Plex Mono (dates,
  figures, badges).
- Status colors: Not Started = slate, In Progress = amber, Blocked = red,
  Done = teal.

## Verification notes
- Commit `6960336` was verified byte-for-byte (SHA-256 match) between the
  locally-generated file and what's on disk in the repo, and confirmed via
  Vercel deployment metadata (commit sha + message) as the live production
  deployment. Local repo, origin/main, and Vercel production are all in sync
  as of this writing.
- Can't fetch live rendered HTML anymore to spot-check content, since
  Vercel Deployment Protection blocks anonymous fetches (see Owner/access
  model). Verification going forward relies on: (a) reading the actual repo
  file directly via Desktop Commander, (b) Vercel deployment metadata
  (commit sha/message matching), not on fetching rendered output.

## Open / not yet done
- No delete-task capability in the UI (add-only, by design so far — ask
  before adding delete).
- No custom domain configured — using Vercel's default subdomains.
