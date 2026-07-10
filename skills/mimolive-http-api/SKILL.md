---
name: mimolive-http-api
description: Control mimoLive (Boinx's macOS live video production app) through its local HTTP API, WebSocket event stream, and MCP endpoint on http://localhost:8989. Use when building integrations or automations that start/stop shows, toggle layers/variants/sources live, control recording and streaming (output destinations), read the program-output image, create layers/sources/outputs, manage Zoom meeting participants, list configured web-service accounts, use data stores, or subscribe to real-time state changes.
---

# mimoLive HTTP API

mimoLive exposes three local interfaces on port **8989**:

- **REST (JSON:API):** `http://localhost:8989/api/v1`
- **WebSocket (live events):** `ws://localhost:8989/api/v1/socket`
- **MCP (for agents):** `http://localhost:8989/mcp`

No auth by default (optional SHA-256 password via the `X-MimoLive-Password-SHA256` header). The server must be enabled in mimoLive → Preferences → Remote Control.

## Read the full reference first

The complete, verified endpoint reference lives in **`mimoLive-API.md`**, next to this skill at the plugin root (`${CLAUDE_PLUGIN_ROOT}/mimoLive-API.md`, i.e. `../../mimoLive-API.md`). **Read it before writing API calls** — it documents every endpoint, the request/response shapes, the type catalogs, the WebSocket protocol, the mlController proxy, and all the gotchas.

## Orientation

- **Find things:** `GET /documents` → each document sideloads its `layers`, `sources`, `output-destinations`, and `layer-sets`.
- **Go live:** `setLive` / `setOff` / `toggleLive` on `documents`, `layers`, `variants`, and `output-destinations`.
- **Create things:** `POST` to `.../layers`, `.../sources`, `.../output-destinations`. Get the required type identifiers from `/layertypes`, `/sourcetypes`, `/outputdestinationtypes`.
- **Built-in outputs:** start/stop record, stream, playout, fullscreen via `GET|POST /documents/{DocID}/outputs/{OutputID}/{action}`.
- **Real-time:** connect to `/api/v1/socket`, send `{"event":"ping"}` every 5 s, and re-fetch a resource whenever you receive an `added`/`removed`/`changed` event for it.
- **Accounts & Zoom:** `GET /accounts` lists the configured web-service accounts; joining a Zoom meeting requires a `zoomaccountname` (since 6.19) — read the names from `/accounts`.

## Critical gotchas

- **Source writes use the nested path** `PATCH /api/v1/documents/{DocID}/sources/{SourceID}` — there is **no** short `/api/v1/sources/{id}` route (it 404s).
- **Built-in output actions toggle** on any verb other than `setLive`/`setOff` — a stray call can *start* a recording.
- Writes need `Content-Type: application/vnd.api+json`. `POST` returns `201`, `DELETE` returns `204`.

See `mimoLive-API.md` for the complete endpoint list, curl examples, and the Zoom / WebSocket / MCP details.
