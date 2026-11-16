---
name: immich
description: 'Immich photo and video management on port 2283 with in-image CUDA machine
  learning on port 3003, deployed on the CachyOS GPU base.

  MUST be invoked before building, deploying, configuring, or troubleshooting the
  immich image.'
---

# immich

Self-hosted photo and video management server — ONE image carrying the web UI, the API,
and the machine-learning backend (face recognition, smart search, OCR).

## Box Properties

| Property | Value |
|----------|-------|
| Image file | `distro-cachyos/box/immich/charly.yml` |
| Base | `nvidia` — the CachyOS GPU base (`cachyos` + agent-forwarding + `nvidia` + `cuda`) |
| Candies | agent-forwarding, nodejs, supervisord, postgresql, vectorchord, redis, immich, dbus |
| Build formats | `pac`, `aur` (pgvector is AUR-only on Arch) |
| Platforms | linux/amd64 |
| Ports | 2283 (web UI + API), 3003 (machine learning) |
| Registry | ghcr.io/opencharly |

The ML backend runs in this image as the supervised `immich-ml` service (priority 40) on
port 3003, and the server reaches it on `http://127.0.0.1:3003`.

## Ports

| Port | Service | Protocol |
|------|---------|----------|
| 2283 | Immich web UI + API | HTTP |
| 3003 | Machine-learning inference API (`/ping`) | HTTP |

## Volumes

| Name | Path | Purpose |
|------|------|---------|
| library | ~/.immich/library | Photo/video storage |
| cache | ~/.immich/cache | Thumbnail cache |
| import | ~/.immich/import | Photo import directory |
| external | ~/.immich/external | External library (no-copy) |
| models | ~/.immich/models | Downloaded ML models (CLIP, face, OCR) |
| pgdata | ~/.postgresql | PostgreSQL data |

## Quick Start

```bash
charly box build immich
charly config immich
charly start immich
# Open http://localhost:2283
```

## GPU

The image is built on the CachyOS GPU base and its ML backend ships ONNX Runtime's
CUDA execution provider. GPU access is not declared per image: the deploy injects
`--device nvidia.com/gpu=all` automatically when the host has an NVIDIA device, so the
same image runs on CPU-only hosts with the ML backend falling back to CPU.

## Key Candies

- `pod-immich` — Immich server, web UI, ML backend, db init, all five volumes
- `pod-postgresql` — database backend
- `layer-vectorchord` — VectorChord for smart search
- `pod-redis` — session/cache backend
- `layer-ffmpeg` — non-free codec support
- `layer-nvidia` / `layer-cuda` — the GPU stack (via the `nvidia` base)

## Verification

- `charly check box immich` — the candy's own build-scope checks (server bundle, web
  build, WASM plugin, geodata set, both wrappers, the ML venv and its CUDA provider)
- After `charly start immich`:
  - `charly status immich` — container running
  - `charly service status immich` — `immich-db-init`, `immich-server`, `immich-ml` RUNNING
  - `curl -s http://localhost:2283/api/server/ping` — `{"res":"pong"}`
  - `curl -s http://localhost:3003/ping` — `pong`

## When to Use This Skill

**MUST be invoked** when the task involves the immich image, Immich photo management,
face recognition, or smart search. Invoke this skill BEFORE reading source code or
launching Explore agents.

## Related

- `/charly-immich:immich-layer` — the candy itself (build recipe, env, services)
- `/charly-image:image` — image family umbrella (composition, build/validate/inspect)
- `/charly-distros:cachyos` — the CachyOS base family and the `nvidia` GPU base
- `/charly-build:build` — the embedded build vocabulary (distros, builders, init-systems)
