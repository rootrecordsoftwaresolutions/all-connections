# Agent operating notes (all-connections)

This repository is a **working snapshot** of every public web surface plus the Ava Ivy desktop client and Python origin. Other agents (Emergent, Cursor, etc.) should treat this README + tree as **full context** for the ecosystem.

## Who Ava is

- **Ava Ivy** is the solar-powered Root Server personality and operator for **Root Record** (real-life / ops / goals) and **RootMC** (Minecraft network).
- Operator email for Ava / Goals: `root@rootrecord.info` (not a user’s Outlook inbox).
- Discord **home** (only unsolicited reports / automations / global chat): channel `1539779979280257054` in guild `1516108585740800042`.
- `#updates` (`1520665313631408251`) is pointer/forward only — do not dump reports there.
- Other Discord rooms: listen; reply when @mentioned or clearly about her. Short RootMC help is OK.
- Slack must not be broken by Discord-home policy (Slack IDs are `C…`, not snowflakes).

## Stack rules (do not invent a new stack)

| Layer | Technology | Not |
|---|---|---|
| Origin / brain | **Python FastAPI** `apps.core.main:app` on `127.0.0.1:8787` | Do not rebuild Node core |
| Ava desktop | **Electron** `ava-desktop/main.mjs` + renderer | Do not replace with a web-only admin |
| Cloudflare Workers | **TypeScript** Wrangler | CF = DNS + Workers + Tunnel, not app hosting for Next |
| Public Next.js | **Vercel** (`vercel/avaivy.cloud`, `vercel/rootrecord.online`) | |
| RootMC marketing/wiki | **Cloudflare Pages** `rootmc/site-pages` → `rootmc.net` | |
| Minecraft plugins | Java/Paper (not in this snapshot) | |

Cheap Cursor model for regular users: `composer-2.5`.

## Never

- Commit `.env`, `credentials.env`, tokens, keypairs, mnemonics, or Firebase admin JSON.
- Force-push `main` / `master`.
- Skip git hooks.
- Point `api.rootmc.net` at Ava FastAPI — that hostname is the **RootMC Worker**. Ava origin is `ava-origin.rootmc.net` / `127.0.0.1:8787`.
- Serve `media/private/` or `documents/docs/vercel-builds/` on the public catalog.
- Treat Solana test funds or leaked mnemonics as live.

## Where to edit what

| Intent | Directory in this repo | Live deploy |
|---|---|---|
| Ava Ivy GUI, lifecycle, ops buttons | `ava-desktop/` | Electron on the OptiPlex (`start-ava-desktop.sh`) |
| HTTP origin, crons, Discord pin, media API | `ava-origin-python/` | uvicorn `:8787` |
| avaivy.cloud | `vercel/avaivy.cloud/` | Vercel |
| rootrecord.online (incl. /goals UI copy) | `vercel/rootrecord.online/` | Vercel |
| Live Goals + membership overlay | `rootrecord-workers/rootrecord-api-goals/` | Worker `g.rootrecord.info` |
| rootmc.net HTML/wiki/home.js | `rootmc/site-pages/` | `wrangler pages deploy` project `rootmc-web` |
| RootMC JSON API | `rootmc/api-worker/` + `rootmc/realm-api/` | Worker `rootmc-api` on `api.rootmc.net` |
| Apex cache / origin failover | `rootmc/site-edge/` | Worker `rootmc-site-edge` (optional; Pages may own apex) |
| Discord + Slack poller | `rootmc/discord-slack-poller/` | `scripts/start-poller.sh` |
| Ava public boards via CF | `rootmc/ava-edge/` | `ava.rootmc.net` |

Primary live monorepo on the SSD is still `/home/ava-core/ava/ava-core-v2` (GitHub `Ava-Core-Dev/ava-core`). After editing here, copy back or PR against that tree — this repo is the **agent-facing combined view**.

## Local origin

```
cd ava-origin-python   # conceptually apps/core
# real host: /home/ava-core/ava/ava-core-v2
.venv/bin/uvicorn apps.core.main:app --host 127.0.0.1 --port 8787
```

Desktop starts origin if `:8787/health` is down (`ava-desktop/lib/avaLifecycle.mjs`). Closing the GUI must **not** kill origin (root server). Watchdog restarts core if health dies while GUI is open.

## DNS / tunnel cheat sheet

- **Cloudflare account RootMC** `f3372b30093435bacc35b69972abeb2e` — `rootmc.net` zone, `rootmc-api`, Pages `rootmc-web`.
- **Cloudflare account Root Record (Goals)** `2b317e91…` — `g.rootrecord.info`, D1 `root-record`.
- **Tunnel (Ava box)** — `ava-origin.rootmc.net`, `ava.rootmc.net`, `site-origin.rootmc.net` → `127.0.0.1:8787`. Do not steal `api.rootmc.net`.
- Homepage activity widget: `home.js` must call a host that is **the Worker**, not NXDOMAIN `api.rootmc.info`. Fallback: `https://rootmc-api.root-337.workers.dev`.
