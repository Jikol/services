# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Homelab Docker services monorepo. Each service lives in its own subdirectory with a `compose.yml`. Services are deployed individually; there is no single root compose file.

## Task Runner

Uses [Taskfile](https://taskfile.dev) (`task` CLI). Secrets loaded from `.env.local` (gitignored; copy `.env.template` to create).

| Command                                   | Alias | Description                          |
| ----------------------------------------- | ----- | ------------------------------------ |
| `task proxy:compose`                      | `p:c` | Deploy proxy-manager                 |
| `task registry:compose`                   | `r:c` | Deploy docker-registry + UI          |
| `task registry:compose BEHIND_PROXY=true` |       | Deploy registry behind proxy network |

CLI_ARGS passthrough:

- _(empty)_ — detached (`-d`), print logs, leave running
- `-- attach` — foreground, tear down on exit
- `-- down` — stop and remove

Example: `task registry:compose -- down`

## Services Without Taskfile Tasks

Run directly from the service directory:

```sh
cd <service-dir>
docker compose up -d
```

Services: `ftp`, `gitea`, `mc-server`, `nginx`, `pi-hole`, `rabbitmq`, `rustdesk`, `share`, `verdaccio`

## Networking

`proxy` is a shared **external** Docker network. Create it once before deploying proxy-dependent services:

```sh
docker network create proxy
```

`proxy-manager` always joins `proxy`. `docker-registry` joins `proxy` only when `BEHIND_PROXY=true` (via `compose.proxy.yml` overlay, which also clears port bindings).

## Adding a New Service

1. Create `<service-name>/compose.yml`.
2. If it needs Task automation, add a public task in `Taskfile.yml` that delegates to the internal `docker:compose` task (follow existing pattern).
3. If it requires secrets, add vars to `.env.template`.
4. If it sits behind the reverse proxy, add a `compose.proxy.yml` overlay that drops port bindings and joins the `proxy` network.

## Environment Variables

`.env.template` documents all required vars. `.env.local` holds real values and is loaded automatically by Taskfile. Variables are marked required with `${VAR:?error}` in compose files — missing vars fail fast at `docker compose up`.

## Services

### Docker Registry UI

@https://raw.githubusercontent.com/eznix86/docker-registry-ui/refs/heads/main/README.md
