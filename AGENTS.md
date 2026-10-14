# Notes for agents working in this repo

## What the app is

A single-process Node.js app: Express serves `views/index.ejs` (EJS templates)
at `/` and static assets from `public/`. Haiku content lives in `haikus.json`,
which `index.js` requires directly. There is **no database, no backend worker
and no external service** — nothing needs credentials.

`web.config` and `process.json` are leftovers from the Azure App Service
sample this template came from; they are not used by the Docker setup.

## Running it (Base44 sandbox)

```bash
docker compose -f docker-compose.base44.yml up -d --build
```

- One service, `web`, on host port **3000** (what the preview shows).
- Runs `node:22` from the bind-mounted working tree, so it is never a prebuilt
  image: `npm ci` runs on startup, then `nodemon --legacy-watch index.js`.
- Polling watch is deliberate — bind mounts here don't reliably emit inotify
  events, so without `--legacy-watch` edits would not restart the server.
- `PORT=3000` is set in compose. Outside the sandbox `index.js` still falls
  back to its own default port.

## Verifying

```bash
curl -fsS http://localhost:3000/ | grep -i haiku   # page renders haikus
docker compose -f docker-compose.base44.yml ps      # web should be healthy
```

`haikus.json` and `views/index.ejs` are the two files to check when the page
looks wrong — Express caches templates only in production, and dev runs here.

## Environment flag

`BASE44_PREVIEW_MODE` is passed into the service but the app currently needs no
sandbox-specific behaviour, so no code reads it. If you ever add one, gate it on
an exact `process.env.BASE44_PREVIEW_MODE === '1'` comparison so unset or any
other value keeps the original behaviour.
