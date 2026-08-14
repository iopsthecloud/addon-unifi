# Context : addon-unifi

A **Home Assistant Supervisor add-on** that packages Ubiquiti's **UniFi Network
Application** so it installs and runs on HA with a single click. The deliverable is not
application code but an **add-on package** — a Docker image plus the Supervisor metadata,
s6 service definitions, and docs that make it installable from the add-on Store.

- Derived from the Home Assistant Community Add-ons layout (Frenck's template); this repo is
  its **own git repository** with its own release/CI workflows under `.github/`.
- **Supervisor descriptor**: `repository.yaml` (repo) + `unifi/config.yaml` (add-on).
- **Rendered docs**: `unifi/DOCS.md` is generated from `unifi/.README.j2`; `README.md` is the
  repo landing page.

## Ubiquitous language

- **Add-on** — the unit the HA **Supervisor** installs. Identified by its **slug** `unifi`;
  its metadata (name, version, ports, options schema, arch) lives in `unifi/config.yaml`.
- **UniFi Network Application** — Ubiquiti's controller software this add-on wraps. Exposed
  to the user as a **web UI** (`webui: https://[HOST]:[PORT:8443]`), not via HA ingress.
- **Repository / Store** — `repository.yaml` marks this as a Supervisor **add-on repository**;
  a user adds its URL to the Store, then installs the `unifi` add-on from it.
- **Base image** — the build starts `FROM ghcr.io/hassio-addons/ubuntu-base` (see
  `unifi/build.yaml` / `Dockerfile`), onto which **Temurin JRE** (Java) and **MongoDB** (the
  controller's datastore) are installed.
- **s6-overlay services** — the container's supervised processes under
  `unifi/rootfs/etc/s6-overlay/s6-rc.d/`: **`init-unifi`** (oneshot setup), **`unifi`**
  (longrun — the controller), and the **`user`** bundle that wires them into the boot order.
- **Options schema** — the user-configurable knobs validated by `config.yaml`'s `schema`:
  `log_level`, plus `memory_max` / `memory_init` (JVM heap sizing for the controller).
- **Ports** — the fixed set the controller needs (8443 web/API, 8080 device comms, 3478 STUN,
  discovery/portal ports…), declared with `ports_description` in `config.yaml`.
- **`backup: cold`** — the add-on is **stopped during backup** so MongoDB's files are
  captured consistently; `map: backup:rw` gives it a backup mount.
- **Arch** — the architectures the image is built for: **`aarch64`** and **`amd64`** only.

## Conventions & gotchas

- Bumping the UniFi/MongoDB/JRE version is a **`Dockerfile` + `config.yaml version`** change,
  then a release via the `.github/workflows/` (release-drafter + deploy). Keep pinned apt
  versions in the `Dockerfile` coherent with the chosen base image.
- Lint/format gates the repo already enforces: `.yamllint`, `.mdlrc` (markdownlint), hadolint
  on the `Dockerfile`. Match them before proposing changes.
- To validate a change in the lab, follow **Part 1 → Part 2** of `HA_TEST_LAB.md`
  (Supervisor add-on flow), not the custom-component flow.

## Decisions

No `docs/adr/` yet. When a non-obvious choice needs recording (e.g. why a base-image or
datastore version was pinned), `/domain-modeling` creates `projets/addon-unifi/docs/adr/`
lazily. Workspace-wide lab/deploy decisions live in the root `docs/adr/`.
