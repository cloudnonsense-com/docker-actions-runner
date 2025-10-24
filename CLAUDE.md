# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Project Overview

Containerized GitHub Actions self-hosted runner on Debian (bookworm-slim) with Docker Compose. Stateless design with automatic lifecycle management.

## Architecture

**Key Design:**
- Runs as root (`RUNNER_ALLOW_RUNASROOT=1`)
- No persistent storage (no volumes/mounts/docker.sock)
- Auto-deregisters on shutdown (prevents orphaned runners)
- Supports org-level and repo-level deployment
- Signal handling for graceful termination

**Components:**
1. `Dockerfile` - Downloads latest runner at build time
2. `entrypoint.sh` - Lifecycle management (registration, signals, deregistration)
3. `compose.yml` - Container orchestration
4. `.env.example` - Configuration template

## Deployment Modes

### Organization-Level Runner
Available to all org repos. Default mode.

**Config:**
- `RUNNER_SCOPE=org` (or omit)
- `GITHUB_ORGANIZATION` - org name
- Token needs `admin:org` scope

**Example:**
```bash
RUNNER_SCOPE=org
GITHUB_ORGANIZATION=my-org
GITHUB_ACCESS_TOKEN=ghp_xxxxx
RUNNER_NAME=org-runner-1
```

### Repository-Level Runner
Available to single repo only.

**Config:**
- `RUNNER_SCOPE=repo`
- `GITHUB_ORGANIZATION` - owner name
- `GITHUB_REPOSITORY` - repo name
- Token needs `repo` scope

**Example:**
```bash
RUNNER_SCOPE=repo
GITHUB_ORGANIZATION=my-org
GITHUB_REPOSITORY=my-repo
GITHUB_ACCESS_TOKEN=ghp_xxxxx
RUNNER_NAME=repo-runner-1
```

## Building and Running

```bash
# Setup environment
cp .env.example .env
# Edit .env with your configuration (see Deployment Modes above)

# Build and start
docker compose up -d --build

# View logs
docker compose logs -f

# Stop (triggers automatic deregistration)
docker compose down
```

## Environment Variables

**Required (in `.env`):**
- `RUNNER_SCOPE` - "org" or "repo" (default: "org")
- `GITHUB_ORGANIZATION` - Org/owner name (no URL)
- `GITHUB_REPOSITORY` - Repo name (only if `RUNNER_SCOPE=repo`)
- `GITHUB_ACCESS_TOKEN` - Token (org: `admin:org`, repo: `repo`)
- `RUNNER_NAME` - Unique runner identifier
- `RUNNER_LABELS` - Labels (default: "self-hosted,linux,x64")

## Signal Handling

entrypoint.sh:75 traps SIGTERM/SIGINT/SIGQUIT/SIGHUP for graceful shutdown. Cleanup function (entrypoint.sh:54-72) gets removal token from GitHub API and calls `config.sh remove --token`.

## GitHub API Integration

**Endpoints by scope:**
- Org: `/orgs/{org}/actions/runners/...`
- Repo: `/repos/{owner}/{repo}/actions/runners/...`

entrypoint.sh:36-45 dynamically constructs endpoints and runner URL based on `RUNNER_SCOPE`.

## Installed Tools

Beyond standard runner:
- **Version Control**: git
- **Shell**: bash, curl, wget, jq, less, vim-tiny, nano
- **Compression**: tar, gzip, unzip
- **Languages**: Node.js, Python 3 (pip, venv)
- **Build**: build-essential, libssl-dev, libffi-dev
- **SSH**: openssh-client, sshpass
- **IaC/Config**: Ansible (2.17.7), Terraform
- **Cloud**: AWS CLI, s3cmd
- **Data**: yq
