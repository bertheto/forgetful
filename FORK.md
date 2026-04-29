# Fork Notice: forgetful-hulkito

This repository is a **private soft-fork** of [ScottRBK/forgetful](https://github.com/ScottRBK/forgetful),
maintained by [@bertheto](https://github.com/bertheto) as a personal safety net and dashboard monorepo.

## Upstream attribution

- **Upstream project**: [ScottRBK/forgetful](https://github.com/ScottRBK/forgetful) — Forgetful MCP, a memory server for AI agents.
- **License**: MIT (see [`LICENCE.md`](./LICENCE.md), preserved intact from upstream — including the British spelling).
- **Copyright**: © 2025 Scott Raisbeck (upstream code unchanged in this fork; additions are in `dashboard/` and top-level fork-governance files).
- **Canonical runtime**: `forgetful-ai` on PyPI is and remains the canonical source. This fork does **not** publish a competing package.

## Purpose of this fork

1. **Safety net (bus factor mitigation)** — upstream has a single maintainer (Scott Raisbeck). If upstream pauses or stops,
   this fork ensures the runtime the Hulkito stack depends on remains patchable.
2. **Dashboard monorepo** — integrates the Svelte 5 dashboard (previously at
   [bertheto/forgetful-dashboard](https://github.com/bertheto/forgetful-dashboard)) into `dashboard/` so server + UI share
   a single sync cadence, CI, and release story.
3. **Upstream-PR staging ground** — branches like `fix/upstream-*` are rebased on `upstream/main` and used to prepare
   patches that go back to upstream. We prefer upstreaming over internalizing maintenance debt.

## Divergence policy

This fork adds **only** the following to upstream's file tree:

- `dashboard/` — Svelte 5 standalone dashboard (imported via `git subtree`, preserves history).
- `FORK.md` — this file.
- `.gitattributes` — `merge=ours` strategy scoped to `dashboard/` and `FORK.md` to minimize sync conflicts.
- `.github/workflows/sync-upstream.yml` — automated weekly upstream sync (opens a PR, never auto-merges).
- `.github/workflows/ci.yml` — fast-CI tuned for the fork (integration + e2e_sqlite + Svelte typecheck, no Postgres E2E).

**We do NOT modify** `app/`, `alembic/`, `main.py`, `pyproject.toml`, `uv.lock`, or any other upstream-owned file
directly in `main`. Changes to those live on `fix/upstream-*` branches and are submitted as PRs to upstream.

## Sync cadence

- **Automated**: `sync-upstream.yml` runs every Monday at 07:00 UTC. It fetches `upstream/main`, detects divergence,
  and opens a PR titled `chore/sync-upstream-YYYY-MM-DD` with the list of upstream commits imported.
  **Auto-merge is disabled** — a human reviewer (Hulkito) must validate and merge.
- **Manual fallback**: `git fetch upstream && git merge upstream/main` (merge-only workflow; rebase breaks `merge=ours`).

## Upstream PRs / Issues tracked

| Kind | Title | Upstream link | Status | Rationale |
|---|---|---|---|---|
| PR | `docs: fix task state enum (todo/doing/waiting/done)` | [#37](https://github.com/ScottRBK/forgetful/pull/37) | OPEN (awaiting review) | DB-level runtime inspection confirms upstream `docs/tool_reference.md` and `app/routes/mcp/tool_metadata_registry.py` documented `pending/in_progress/completed/blocked/cancelled` while live DB has canonical `todo/doing/waiting/done/cancelled` (`TaskState` enum in `app/models/plan_models.py`). |
| Issue | `Graph API: /api/v1/graph doesn't expose skill, plan, or task nodes` | [#38](https://github.com/ScottRBK/forgetful/issues/38) | OPEN (awaiting maintainer input) | `/api/v1/graph` validator rejects `skill`/`plan`/`task` but `SubgraphNode.type` Literal already supports them. Client dashboard works around via N+M+P calls. Issue proposes 3-phase design + 4 open questions to Scott before PR. Posted 2026-04-21. |
| PR | `feat(api): Phase 1 — expose skill nodes on /api/v1/graph` | (future, branch `feat/upstream-graph-skills`) | BLOCKED (pending issue response) | Will unblock `dashboard/src/lib/api/graph.ts::fetchPlanningAndSkills` workaround when Phase 1+2+3 are merged. |

This table is updated whenever a PR or issue is opened, merged, closed, or changes status upstream. Keep the status in sync with the `Upstream Contributions` epic (`212`) in the `forgetful-hulkito` kanban project.

## Project tracking

This fork is tracked across two systems:

- **Kanban (kanban-mcp)** — project `forgetful-hulkito`, `project_id=31f1a8c54cc776a2`, `project_dir=C:\work\cursor\perso\forgetful-hulkito`.
  - Seed epics: `210` Fork Setup & Governance, `211` Dashboard Integration, `212` Upstream Contributions
  - Story prefix: `FF-XX`, epic prefix: `EPIC-FF-XX`, always-tag: `forgetful-fork`
  - Tag taxonomy: `workstream` (`ws1`-`ws5`), `status` (`done`/`blocked`), `category` (`docs`/`ci-sync`/`upstream-pr`/`upstream-issue`/`fork-governance`), `graph-phase` (`phase-1`/`phase-2`/`phase-3`)
- **Forgetful memory** — `project_id=13`, name `forgetful-hulkito`, type `open-source`. Reusable technical patterns captured here (soft-fork governance, upstream PR recipes, dashboard integration gotchas, SQLite-WAL constraints).

Autonomy rule `~/.cursor/rules/forgetful-hulkito-autonomy.mdc` grants the agent permission to auto-manage epics/stories/tags/status transitions and to enforce doc-sync (FORK.md / `dashboard/README.md` / CHANGELOG.md) on every session touching this repo. Kill switch: rename file to `.disabled` or add `DISABLED: true` marker.

Active-context snapshot: `~/.cursor/active-contexts/forgetful-hulkito.md`.

## Rollback strategy

Triggers that would justify unwinding this fork:

- Upstream publishes an integrated dashboard — this fork becomes redundant.
- Maintenance exceeds 2h/month for 3 consecutive months.
- Upstream architectural rewrite makes merges ingérable.
- Scott disappears — fork stays but moves to **archive mode** (no new divergence).

High-level procedure: `gh repo archive bertheto/forgetful-hulkito` → migrate open PRs to gist → restore
`~/.cursor/tools/forgetful-dashboard/` as dashboard source → update `forgetful-mcp` skill → mark
"soft-fork pattern" memory obsolete. Cost: 1-2h.

### Rollback procedure (detailed scenarios)

Three concrete scenarios, in order of likelihood:

1. **Scenario A — Upstream publishes a dashboard / fork redundant** (most likely):
   - `git fetch upstream && git switch main`
   - `git reset --hard upstream/main` (discards all fork-only files; `.gitattributes`, `FORK.md`, `dashboard/` disappear)
   - `gh repo archive bertheto/forgetful-hulkito` (keeps repo read-only for audit)
   - Update Forgetful memory `#548` (originally `#542`, recreated 2026-04-21 post-closing to restore embedding — `#542` is obsolete, superseded_by=548) to mark `project_id=13` as archived (`metadata.archived_at`)
   - Update Hulkito stack to consume `ScottRBK/forgetful` directly (revert docker-compose, CI refs, skill pointers)
   - Remove Kanban project `forgetful-fork` (kanban-mcp) after archiving its stories
   - Cost: ~2h (stack refactor is the bulk)

2. **Scenario B — Upstream abandoned, `forgetful-hulkito` becomes canonical**:
   - Keep fork active; rename default branch if needed to signal canonical status
   - Optional: publish to PyPI under a different name (e.g. `forgetful-hulkito` — **do NOT reuse `forgetful-ai` to avoid typosquatting/competing**)
   - Update `FORK.md` header to "Hard fork — upstream archived YYYY-MM-DD"
   - Disable `sync-upstream.yml` (rename to `.disabled` or add `if: false`)
   - Fork-fast guards in `e2e.yml`/`build.yml`/`publish.yml` must be **removed** (this fork now IS the upstream for the Hulkito stack)
   - Cost: ~4h (PyPI setup + CI rework)

3. **Scenario C — Divergence too large, rebuild hot from scratch**:
   - New private repo `forgetful-hulkito-v2` from upstream/main HEAD at time T
   - Re-apply `dashboard/` via `git subtree add --prefix=dashboard <dashboard-origin> main --squash`
   - Re-apply FORK.md, `.gitattributes`, `sync-upstream.yml` (copy from old repo)
   - `gh repo archive bertheto/forgetful-hulkito` (old repo kept for history)
   - Lessons-learned memory in Forgetful (`category=decision`, link to `#548` (ex-`#542`, obsoleted post-closing), `#545`)
   - Cost: ~3h (mostly testing the new merge=ours baseline)

## Contact / Questions

Anything about this fork (not about upstream behavior) goes to the private issue tracker of this repo. Upstream
questions go to [ScottRBK/forgetful/issues](https://github.com/ScottRBK/forgetful/issues).
