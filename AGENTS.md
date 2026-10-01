# Base44 Development Environment

## Project Overview
This is the **Expo SDK monorepo** (`@expo/expo`). The runnable web app is the
**docs site** at `docs/` — a Next.js 15 (pages router) site for docs.expo.dev.

## What Runs in the Preview
Only the `docs/` Next.js app runs. The rest of the monorepo (native apps, Expo Go,
packages) targets iOS/Android and cannot run in a browser preview.

## docs/ App Setup
- **Package manager**: Yarn Berry 4.9.1 (`packageManager` field in `docs/package.json`).
  The binary is vendored at `docs/.yarn/releases/yarn-4.9.1.cjs`; `.yarnrc.yml`
  sets `nodeLinker: node-modules`.
- **Standalone**: `docs/` is NOT part of the root yarn workspace (workspaces are
  `apps/*`, `packages/*`, `packages/@expo/*`). It has its own `yarn.lock`.
- **Static resources**: `yarn generate-static-resources` writes
  `docs/public/static/constants/{versions,navigation}.json` from
  `docs/constants/{versions,navigation}.js`. Also runs via `postinstall`.
- **copy-latest**: `yarn copy-latest` copies `pages/versions/v<version>/` to
  `pages/versions/latest/`. The current version is in `docs/package.json` (`54.0.0`).
- **Dev server**: `next dev -p 3002` by default; overridden to port 3000 in compose.
- **Sentry**: `withSentryConfig` wraps the config; `SENTRY_AUTH_TOKEN` is optional
  in dev mode (only needed for source map upload during build).

## Docker Compose
`docker-compose.base44.yml` runs a single `docs` service:
- `node:22` base image, repo bind-mounted at `/app`, working dir `/app/docs`.
- `corepack enable && yarn install && yarn generate-static-resources && next dev -p 3000 -H 0.0.0.0`.
- Healthcheck: `curl -sf http://localhost:3000/` with 180s startup grace.
- `allowedDevOrigins` added to `docs/next.config.ts` for the preview origin.

## Verification
```sh
docker compose -f docker-compose.base44.yml up -d --build
docker compose -f docker-compose.base44.yml ps
curl -sf http://localhost:3000/   # should serve the docs homepage
```

## No External Secrets Required
The docs dev server needs no external credentials. `SENTRY_AUTH_TOKEN` is optional.
