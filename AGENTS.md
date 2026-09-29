# AGENTS.md — pod-immich

Standalone candy repo for the `immich` candy — the self-hosted photo and video
management server on `2283`. The candy lives in `charly.yml` at the repo root.

Canonical files:

- `charly.yml` — the `immich:` candy entity (description, `require`, `env`,
  `distro`, `port`, `env_provide`, `env_accept`, `secret_accept`, `volume`,
  `secret`, `route`, `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-immich:immich` — the owning skill: box properties, the candy stack,
  the volumes, and verification. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-immich:immich-ml` — the CUDA ML sibling for face recognition and smart
  search.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `env_provide` / `secret_accept`, service declarations).
- `/charly-build:secrets` — the credential store backing the `secret_accept` /
  `secret` entries.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the built server bundle, the two wrappers, the `libvips` image
  library, the extension-seeding init SQL, the mounted library dir, the running
  `immich-server` service, and the live `/api/server/ping` `200` returning
  `pong`.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `immich:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- The Immich version pin is the `IMMICH_VERSION` env/var; keep it in step across
  the source fetch and the server build.
- The DB-migration and server-start wrappers are authored inline in the `plan:`.
  Keep the extension list (`vector` / `vchord` / `earthdistance`) in step with the
  `pod-postgresql` / `vectorchord` dependencies.
- The four volumes are the persistent stores; a change to their paths must move
  with the server's `IMMICH_MEDIA_LOCATION`.
- The `skill:` entity is the source for `/charly-immich:immich`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
