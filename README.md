# pod-immich

The `immich` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships the [Immich](https://immich.app) self-hosted
photo and video management server.

## What it provides

Builds Immich **v3.4.2** from source into one image: the Node server into
`/opt/immich/server`, the web UI into `/build/www`, the plugin-core WASM plugin and
the reverse-geocoding geodata into `/build/`, plus the Python machine-learning
virtualenv (ONNX Runtime with the CUDA execution provider) that serves face
recognition, smart search and OCR on port `3003`. It installs a PostgreSQL-aware
startup wrapper and an idempotent DB-migration wrapper, and seeds Postgres with the
`vector` / `vchord` / `cube` / `earthdistance` extensions before serving the
photo+video API on port `2283`. Library, cache, import, external media and the ML
model cache live under `~/.immich`.

The machine-learning backend is part of THIS candy. The former sibling repo
`pod-immich-ml` (the separate `immich-ml` candy/image) is retired: its functionality
lives here.

| Property | Value |
|---|---|
| Services | `immich-db-init` (`immich-db-migrate.sh`, priority 15), `immich-server` (`immich-server-start.sh`, priority 30), `immich-ml` (`python -m immich_ml`, priority 40) |
| Ports | `2283` (web UI + API), `3003` (machine learning) |
| Route | `immich.localhost` |
| Requires | `layer-supervisord`, `layer-nodejs`, `pod-postgresql`, `pod-redis`, `layer-ffmpeg` |
| Volumes | `library` `~/.immich/library`, `cache` `~/.immich/cache`, `import` `~/.immich/import`, `external` `~/.immich/external`, `models` `~/.immich/models` |
| env_provide | `IMMICH_API_URL` (`http://{{.ContainerName}}:2283/api`) |
| secret_accept | `IMMICH_API_KEY` (`charly/api-key/immich`) |

The `immich-db-init` service is `restart: no` — it migrates and seeds once per
start; `immich-server` waits on `pg_isready` then execs the built server
(`node /opt/immich/server/dist/main.js` — the ESM entry needs its extension).

## How to use it

```bash
charly box build immich
charly config immich
charly start immich
# open http://localhost:2283
```

Then generate an API key in the Immich admin UI and store it once for every
consumer (`openwebui`, `hermes`, and this box share the key path):

```bash
charly secrets set charly/api-key/immich <key>
```

## Layout

- `charly.yml` — the `immich:` candy entity plus its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skills: `/charly-immich:immich-layer` — this candy (build recipe, env,
  services, volumes); `/charly-immich:immich` — the image that composes it
  (`distro-cachyos/box/immich-cdi`, CachyOS GPU base) and its verification.
- `/charly-infrastructure:postgresql` / `/charly-infrastructure:vectorchord` /
  `/charly-infrastructure:redis` — the backing data services.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
