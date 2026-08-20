# Self-Hosted GitHub Actions Runner

Credit to: https://github.com/youssefbrr/self-hosted-runner/

---

## Quick Start

```sh
git clone https://github.com/lhtran-homelab/gh-runner-container.git
cd self-hosted-runner
cp .env.example .env        # fill in REPO, REG_TOKEN
```

**Linux (amd64/arm64)**
```sh
docker-compose -f ./docker-compose.yml up -d
```

> **REG_TOKEN expires after 1 hour.** Generate a fresh one from
> GitHub → Settings → Actions → Runners → "New self-hosted runner" before each deploy.

---

## Pre-built Image vs Local Build

Both variants support pre-built images from GHCR. By default, `docker-compose up` pulls the pre-built image — no build step required.

| Variant | Image | Tag |
|---------|-------|-----|
| **Linux (x64)** | `ghcr.io/lhtran-homelab/gh-runner-container` | `latest` |

| Mode | How | When to use |
|------|-----|-------------|
| **Pre-built** (default) | Just run `docker-compose up` | Quick setup, no customization needed |
| **Local build** | Uncomment `build: .` in the compose file | Custom Dockerfile changes, runner version overrides |

---

## Features

- **Zero-config start** — set 3 env vars and run
- **Clean shutdown** — SIGINT/SIGTERM deregisters the runner automatically
- **Scalable** — Linux defaults to 2 replicas; tune with `deploy.replicas`
- **Ephemeral mode** — run once and self-destruct (`EPHEMERAL=true`)
- **Docker-in-Docker** — macOS image mounts the Docker socket for nested builds
- **GitHub CLI** — `gh` pre-installed from official repos on both variants
- **Docker CLI** — official Docker CE CLI with buildx and compose plugins
- **Healthchecks** — built-in `pgrep run.sh` health monitoring on both variants

---

## Configuration

Copy `.env.example` to `.env` and set your values. The `.env` file is gitignored.

### Required

| Variable | Description |
|----------|-------------|
| `REPO` | `owner/repo` for repo-level or `owner` for org-level runners |
| `REG_TOKEN` | Registration token from GitHub Settings (expires in 1 hour) |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `NAME` | random | Display name shown in GitHub Actions UI |
| `LABELS` | _(none)_ | Comma-separated labels, e.g. `self-hosted,linux,x64,gpu` |
| `RUNNER_GROUP` | _(default)_ | Runner group name — org/enterprise only |
| `WORK_DIR` | `_work` | Workspace directory inside the container |
| `EPHEMERAL` | `false` | `true` → deregister after one job |
| `DISABLE_AUTO_UPDATE` | `false` | `true` → prevent runner self-updates |

---

## Registering to an Organization

Set `REPO` to just the org name:

```env
REPO=my-org
```

The runner will register at org level and be available to all repositories in that org.

---

## Workflows Example

Reference your runner in any workflow file:

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux]
    steps:
      - uses: actions/checkout@v4
      - run: echo "Running on self-hosted runner"
```

Use your custom labels to target specific runners:

```yaml
runs-on: [self-hosted, linux, gpu]
```

---

## Scaling

Adjust replicas in `docker-compose.yml` under `deploy.replicas`. GitHub will distribute jobs across all registered runners automatically.

```yaml
deploy:
  replicas: 4       # spin up 4 concurrent runners
  resources:
    limits:
      cpus: '0.5'
      memory: 512M
```

---

## Troubleshooting

**Runner doesn't appear in GitHub Settings**
- Check `REG_TOKEN` — it expires after 1 hour. Generate a new one.
- Verify `REPO` format: `owner/repo` (no leading slash, no trailing slash).

**Logs**
```sh
docker-compose -f docker/linux/docker-compose.yml logs -f
```

**Health status**
```sh
docker-compose -f docker/linux/docker-compose.yml ps
```

**Runner stuck / won't deregister**
```sh
docker-compose -f docker/linux/docker-compose.yml down
```
`down` sends SIGTERM → `start.sh` cleanup → runner deregisters cleanly.

---

## License

[MIT](LICENSE) — use freely, attribution appreciated.
