# CLI launcher + thin web UI, single Node process

OpusLoomV2 is shipped as a CLI binary that launches and supervises a
Node server. The server serves both the JSON API (Pi Durable over REST
+ SSE) and the Vue 3 web UI as static assets. One process, one port;
one `node bin/opusloom.js` runs the whole stack.

**Why.** Solo-dev friendly — one binary, one supervisor, no separate
web server to deploy. Pi Pocket's exact pattern (proven in their
`bin/pi-pocket.js` + `src/launcher/main.ts` + `src/server/main.ts`).
Multi-workspace switcher is v2+; this design covers the v1 single-
workspace case cleanly.

**Components:**
- `bin/opusloom.js` — Node entry, version check, spawns launcher.
- `src/launcher/main.ts` — access menu (local/LAN/Cloudflare/Tailscale),
  supervises the server, holds the tunnel up across server restarts.
- `src/server/main.ts` — Express + Pi Durable; serves `/v1/*` (JSON API
  + SSE) and `/app/*` (Vue 3 static build).
- `web/` — Vue 3 + Vite frontend, builds to `web/dist/`, served by
  the Node server.

**Consequences.** A supervisor process (the launcher) can restart the
server on crash; the server's logs are streamed to the launcher's
terminal. The Tauri shell is rejected for v1 — adds a Tauri runtime
dependency for no v1 win; the Vue-in-browser pattern is sufficient.
Multi-workspace (v2) may need either multi-launcher (heavy) or
workspace switching in the Vue app (lighter).
