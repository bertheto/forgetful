# Forgetful Dashboard (Svelte 5)

A standalone Svelte 5 + Vite + TypeScript dashboard for the [Forgetful MCP](https://github.com/ScottRBK/forgetful)
memory server. This dashboard is integrated into `forgetful-hulkito` (this repo) as a subtree; it previously
lived at [bertheto/forgetful-dashboard](https://github.com/bertheto/forgetful-dashboard).

## Features

- **Interactive memory graph** (vis-network 10) with project/tag filtering, clustering, node limit, sort order
- **Stats panel** (Chart.js 4) — memory type distribution, activity-over-time
- **Memory table** — triable/filtrable, inline edit, related memories
- **Briefing tab** — daily briefing memory, pinned items
- **Activity stream** — SSE-powered live feed of memory/task/plan mutations
- **Admin modal** — import/export/backup
- **OAuth** — GitHub OAuth via FastMCP (canonical upstream auth path)

## Prerequisites

- **Node.js** 22+ and **npm** 10+ (the `Dockerfile` uses `node:22-alpine`).
- **Forgetful MCP server running on port 8020** — this dashboard does **not** ship a server.
  - Recommended: run the server **on the host** (not in Docker) to avoid SQLite WAL contention
    (see upstream memory-bus note in `FORK.md` and project memory #394).
  - The launcher `~/.cursor/tools/forgetful-dashboard/start-forgetful-http.ps1` handles this on Windows.

## Compatibility matrix

| Dashboard version | `forgetful-ai` min | `forgetful-ai` max tested | Notes |
|---|---|---|---|
| v1.0 (current) | 0.3.0 | 0.3.2 | Baseline post-soft-fork. Depends on `fetchPlanningAndSkills` client-side workaround until upstream PR #1 lands. |
| v1.1 (planned) | 0.3.3 or later | latest | To be cut if upstream merges `feat(api): include skills/plans/tasks in /api/v1/graph`. |

This table is maintained manually when a new `forgetful-ai` release is validated end-to-end. See `FORK.md` for PR status.

## Quickstart — dev mode (hot reload)

```powershell
# Terminal 1: start the MCP server on the host
cd ~/.cursor/tools/forgetful-dashboard
./start-forgetful-http.ps1

# Terminal 2: start the Svelte dev server
cd dashboard
npm install
npm run dev  # serves http://localhost:8021 with /api proxy to :8020
```

## Production — Docker (dashboard only, server stays on host)

```powershell
# Host-side server (mandatory)
./start-forgetful-http.ps1  # in ~/.cursor/tools/forgetful-dashboard/

# Dashboard container
cd dashboard
docker compose -f docker-compose.dashboard.yml up -d --build
# -> http://localhost:8021
```

`docker-compose.dashboard.yml` uses `host.docker.internal:8020` for the API, and enables `extra_hosts` for
Linux compatibility.

## Architecture notes

- **Why npm + Vite bundling instead of CDN + SRI?** Bundling produces deterministic deliverables
  (`package-lock.json` pins every transitive), is compatible with `npm audit` / Snyk, and avoids runtime CDN
  availability risks. SRI hashes add value only when the browser fetches the module directly — with Vite
  everything is bundled at build time. This is a **deviation from the original plan section "deps via CDN + SRI"**
  that was decided once the existing Svelte repo was imported (the plan assumed a from-scratch rebuild).
- **`merge=ours` applies to the whole `dashboard/`** — upstream will never touch this folder, but we belt-and-brace
  via `.gitattributes` in case a future upstream introduces a `dashboard/` of its own.
- **Auth** — localhost-only deployment. OAuth flow is GitHub via FastMCP; tokens stored in `localStorage`.
  Do not expose the dashboard publicly without adding TLS + CSRF guards upstream.

## Known issues / roadmap

| Item | Status | Ref |
|---|---|---|
| `/api/v1/graph` missing `skill`/`plan`/`task` node types | Workaround in `src/lib/api/graph.ts` (`fetchPlanningAndSkills`) | Upstream PR #1 pending (see `FORK.md`) |
| Upstream `docs/tool_reference.md` documents `pending/in_progress/completed` states; live schema uses `todo/doing/waiting/done` | Our skills + memory #261 use the correct runtime values | Upstream PR #2 pending (doc fix, see `FORK.md`) |
| Activity SSE was "undocumented" in 0.3.0 | Now documented in upstream `docs/api_reference.md`. Workaround retired. | N/A — resolved |

## Tests / checks

```powershell
npm run check   # svelte-check + tsc strict mode
npm run build   # Vite production build (outputs to dist/)
```

CI (`.github/workflows/ci.yml` at repo root) runs `npm ci && npm run build && npm run check` on every PR.

## License

MIT — inherited from upstream `ScottRBK/forgetful` (see repo root `LICENCE.md`). Dashboard code is © Hulkito
where new; upstream code is © Scott Raisbeck.
