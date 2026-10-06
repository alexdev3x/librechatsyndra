# LibreChat — Base44 Development Environment

## Overview
LibreChat is a full-stack AI chat application (Node.js/Express API + React/Vite client + MongoDB + MeiliSearch).
The repo is an npm workspaces monorepo: `api/`, `client/`, and `packages/*`.

## Running the app
```sh
docker compose -f docker-compose.base44.yml up -d
```
The app is served on port 3000 (Vite dev server). The API runs on port 3080 inside the container and is proxied by Vite.

## Architecture
- **app** (node:20): Single container running both the API (nodemon on :3080) and client (Vite on :3000).
  - Startup: `npm ci` → `npm run build:packages` → create dummy `client/dist/index.html` → start API in background → `exec npm run frontend:dev`
  - Source is bind-mounted; `node_modules` uses a named volume.
  - The API reads `client/dist/index.html` at startup (line 65 of `api/server/index.js`). A dummy file is created in the startup command so the API boots in dev mode (where Vite serves the real client).
- **mongodb** (mongo:8.0): No auth. Data in named volume.
- **meilisearch** (v1.12.3): Search engine. Data in named volume.

## Key details
- Workspace packages (`packages/data-provider`, `packages/api`, `packages/data-schemas`, `packages/client`) are TypeScript and must be built (`npm run build:packages`) before the API and client can use them.
- The Vite dev server proxies `/api` and `/oauth` to the backend. `HOST=0.0.0.0` is used for both binding and proxy target (works because both processes are in the same container).
- AI provider API keys default to `user_provided` — users enter keys in the UI. No external credentials needed to boot.
- `ALLOW_REGISTRATION=true` is set in `.env.base44-defaults` so new users can register.
- Default secrets (CREDS_KEY, JWT_SECRET, etc.) are from `.env.example` and will log warnings — replace them for production.

## Files
- `docker-compose.base44.yml` — dev compose (the runbook)
- `.env.base44-defaults` — development env defaults (overridden by `/run/base44/app.env`)
- `.base44/environment.json` — platform metadata

## Verification
```sh
curl -sf http://localhost:3000/              # Vite dev server (HTML)
curl -sf http://localhost:3000/api/config     # API through Vite proxy (JSON)
```
