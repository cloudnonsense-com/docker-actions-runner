# GitHub Actions Self-Hosted Runner (Docker)

Lightweight, containerized GitHub Actions runner with automatic lifecycle management. Supports both organization and repository-level deployment.

## Features

- **Flexible Deployment**: Organization-level or repository-level runners
- **Auto-Deregistration**: Prevents orphaned runners on shutdown
- **Stateless**: No persistent storage or volumes
- **Signal Handling**: Graceful termination (SIGTERM, SIGINT, SIGQUIT, SIGHUP)
- **Lightweight**: Debian bookworm-slim base
- **Always Current**: Downloads latest runner at build time

## Prerequisites

- Docker and Docker Compose
- **GitHub Personal Access Token with CORRECT scope (critical!):**
  - **Organization-level runners**: Token MUST have `admin:org` scope
  - **Repository-level runners**: Token MUST have `repo` scope
  - ⚠️ **Wrong scope = 404 error during registration**

## Deployment Modes

### Organization-Level (Default)
Available to all repos in the organization.

```bash
cp .env.example .env
# Edit .env:
RUNNER_SCOPE=org
GITHUB_ORGANIZATION=your-org
GITHUB_ACCESS_TOKEN=ghp_xxxxx  # ⚠️ MUST have admin:org scope
RUNNER_NAME=org-runner-1
```

### Repository-Level
Available to a single repository only.

```bash
cp .env.example .env
# Edit .env:
RUNNER_SCOPE=repo
GITHUB_ORGANIZATION=your-org
GITHUB_REPOSITORY=your-repo
GITHUB_ACCESS_TOKEN=ghp_xxxxx  # ⚠️ MUST have repo scope (NOT admin:org)
RUNNER_NAME=repo-runner-1
```

## Quick Start

```bash
# 1. Configure (see Deployment Modes above)
cp .env.example .env
# Edit .env with your settings

# 2. Build and start
docker compose up -d --build

# 3. Verify registration
# Org: GitHub org settings → Actions → Runners
# Repo: GitHub repo settings → Actions → Runners

# 4. View logs
docker compose logs -f

# 5. Stop (auto-deregisters)
docker compose down
```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `RUNNER_SCOPE` | No | `org` | Deployment scope: `org` for organization-level or `repo` for repository-level |
| `GITHUB_ORGANIZATION` | Yes | - | GitHub organization/owner name (without URL prefix) |
| `GITHUB_REPOSITORY` | Conditional | - | Repository name (required when `RUNNER_SCOPE=repo`) |
| `GITHUB_ACCESS_TOKEN` | Yes | - | Personal access token (scope depends on deployment mode) |
| `RUNNER_NAME` | Yes | - | Unique identifier for this runner instance |
| `RUNNER_LABELS` | No | `self-hosted,linux,x64` | Comma-separated labels for workflow targeting |

### Creating a GitHub Personal Access Token

GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)

**⚠️ CRITICAL: Token scopes MUST match deployment mode**

| Deployment Mode | Required Scope | Common Mistake |
|----------------|----------------|----------------|
| Organization (`RUNNER_SCOPE=org`) | ✅ `admin:org` | Using `repo` scope instead |
| Repository (`RUNNER_SCOPE=repo`) | ✅ `repo` | Using `admin:org` scope instead |

**Symptom of wrong scope:** Registration fails with `404 Not Found` error

## How It Works

1. Validates environment and determines deployment scope
2. Constructs appropriate GitHub API endpoints (org or repo)
3. Fetches registration token and registers runner
4. Runs workflow jobs until termination signal received
5. Auto-deregisters on shutdown to prevent orphaned runners

## Installed Tools

Pre-installed for CI/CD workflows:

- **Version Control**: git
- **Shell & Utilities**: bash, curl, wget, jq, less, vim-tiny, nano
- **Compression**: tar, gzip, unzip
- **Languages**: Node.js, Python 3 (with pip and venv)
- **Build Tools**: build-essential, libssl-dev, libffi-dev
- **SSH**: openssh-client, sshpass
- **IaC & Config**: Ansible (2.17.7), Terraform
- **Cloud**: AWS CLI, s3cmd
- **Data**: yq (YAML/JSON processor)

## Troubleshooting

**❌ Error: "404 Not Found" during registration:**
- **MOST COMMON CAUSE**: Wrong token scope!
  - Org mode needs `admin:org` scope (NOT `repo`)
  - Repo mode needs `repo` scope (NOT `admin:org`)
- Verify token scopes: `curl -I -H "Authorization: token YOUR_TOKEN" https://api.github.com/user`
  - Look for `x-oauth-scopes` header in response
- Create new token with correct scope if needed

**Runner fails to start:**
- Verify `GITHUB_ACCESS_TOKEN` has correct scope (see above)
- Check `GITHUB_ORGANIZATION` is exact (case-sensitive)
- For repo mode: ensure `RUNNER_SCOPE=repo` and `GITHUB_REPOSITORY` are set
- Review logs: `docker compose logs`

**Runner not appearing:**
- Check names are correct (not full URLs)
- Confirm token hasn't expired

**Runner offline:**
- Check container status: `docker compose ps`
- View logs: `docker compose logs`

## Security

- **Never commit `.env`**: Contains sensitive tokens
- **Minimal scopes**: Use only required token permissions
- **Runs as root**: Simplifies setup; consider non-root for production
- **No persistent storage**: Stateless by design

## License

This project is provided as-is for use with GitHub Actions self-hosted runners.
