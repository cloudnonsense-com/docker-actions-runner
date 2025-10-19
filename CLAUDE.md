# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a containerized GitHub Actions self-hosted runner designed to run continuously without persistent storage. The runner is built on Debian (bookworm-slim) and uses Docker Compose for deployment.

## Architecture

**Key Design Decisions:**
- Runs as root using `RUNNER_ALLOW_RUNASROOT=1` to simplify the setup
- No persistent storage (no volumes, no mounts, no docker.sock)
- Signal handling for graceful shutdown and automatic deregistration from GitHub Actions
- Prevents orphaned runners by deregistering on container termination

**Components:**
1. `Dockerfile` - Debian-based image that downloads the latest GitHub Actions runner at build time
2. `entrypoint.sh` - Handles runner lifecycle: registration, signal trapping, and deregistration
3. `compose.yml` - Orchestrates the container with environment variable injection
4. `.env.example` - Template for required configuration

## Building and Running

```bash
# Setup environment
cp .env.example .env
# Edit .env with your GitHub organization, access token, runner name, and labels

# Build and start
docker compose up -d --build

# View logs
docker compose logs -f

# Stop (triggers automatic deregistration)
docker compose down
```

## Environment Variables

Required variables (defined in `.env`):
- `GITHUB_ORGANIZATION` - GitHub org name (not the full URL)
- `GITHUB_ACCESS_TOKEN` - Personal access token with `repo` and `admin:org` scopes
- `RUNNER_NAME` - Unique identifier for this runner instance
- `RUNNER_LABELS` - Comma-separated labels (defaults to "self-hosted,Linux,X64")

## Signal Handling

The entrypoint script (entrypoint.sh:51) traps SIGTERM, SIGINT, SIGQUIT, and SIGHUP to ensure the runner deregisters itself before shutdown. The cleanup function (entrypoint.sh:30-48) obtains a removal token from the GitHub API and calls `config.sh remove --token` to deregister the runner without interactive prompts.

## GitHub API Integration

The runner uses the GitHub API to obtain a registration token (entrypoint.sh:54-63) before registering with the organization. The access token must have sufficient permissions to manage self-hosted runners.

## Installed Tools

The Docker image includes the following tools beyond the standard GitHub Actions runner:
- **Version Control**: git
- **Shell utilities**: bash, curl, wget, jq, less
- **Compression**: tar, gzip, unzip
- **Programming**: Node.js, Python 3 (with pip and venv)
- **Build tools**: build-essential, libssl-dev, libffi-dev
- **SSH**: openssh-client, sshpass
- **Configuration Management**: Ansible (ansible-core 2.17.7, ansible 10.7.0)
- **Data processing**: yq (YAML/JSON processor)
