# pod-immich

The `immich` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships the [Immich](https://immich.app) self-hosted
photo and video management server.

## What it provides

Builds the Immich Node server from source with pnpm into `/opt/immich/server`,
installs a PostgreSQL-aware startup wrapper and an idempotent DB-migration
wrapper, and seeds Postgres with the `vector` / `vchord` / `earthdistance`
extensions before serving the photo+video API on port `2283`. Library, cache,
import, and external media live under `~/.immich`.

| Property | Value |
|---|---|
| Services | `immich-db-init` (`immich-db-migrate.sh`, priority 15), `immich-server` (`immich-server-start.sh`, priority 30) |
| Port | `2283` (web UI + API) |
| Route | `immich.localhost` |
| Requires | `layer-supervisord`, `layer-nodejs`, `pod-postgresql`, `pod-redis`, `layer-ffmpeg` |
| Volumes | `library` `~/.immich/library`, `cache` `~/.immich/cache`, `import` `~/.immich/import`, `external` `~/.immich/external` |
| env_provide | `IMMICH_API_URL` (`http://{{.ContainerName}}:2283/api`) |
| secret_accept | `IMMICH_API_KEY` (`charly/api-key/immich`) |

The `immich-db-init` service is `restart: no` — it migrates and seeds once per
start; `immich-server` waits on `pg_isready` then execs the built server.

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

- Owning skill: `/charly-immich:immich` — the box properties, the candy stack,
  the volumes, and verification.
- `/charly-immich:immich-ml` — adds CUDA ML for face recognition and smart search.
- `/charly-infrastructure:postgresql` / `/charly-infrastructure:vectorchord` /
  `/charly-infrastructure:redis` — the backing data services.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
