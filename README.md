# GitHub Actions Self-Hosted Runner (Docker)

A lightweight, containerized GitHub Actions self-hosted runner that runs continuously and automatically manages its lifecycle. Built on Debian (bookworm-slim) for minimal resource usage.

## Features

- **Continuous Operation**: Runner stays active and processes jobs indefinitely
- **Automatic Deregistration**: Cleanly removes itself from GitHub Actions on shutdown to prevent orphaned runners
- **Stateless Design**: No persistent storage required (no volumes, no mounts)
- **Signal Handling**: Gracefully handles termination signals (SIGTERM, SIGINT, SIGQUIT, SIGHUP)
- **Lightweight**: Based on Debian (bookworm-slim)
- **Latest Runner**: Automatically downloads the latest GitHub Actions runner version at build time

## Prerequisites

- Docker and Docker Compose installed
- GitHub organization with permissions to add self-hosted runners
- GitHub Personal Access Token with the following scopes:
  - `repo`
  - `admin:org` (for managing self-hosted runners)

## Quick Start

1. **Clone and configure**:
   ```bash
   cp .env.example .env
   ```

2. **Edit `.env` file** with your values:
   ```bash
   GITHUB_ORGANIZATION=your-organization-name
   GITHUB_ACCESS_TOKEN=ghp_your_token_here
   RUNNER_NAME=my-runner-1
   RUNNER_LABELS=self-hosted,Linux,X64,docker
   ```

3. **Build and start the runner**:
   ```bash
   docker compose up -d --build
   ```

4. **Verify the runner is registered**:
   - Go to your GitHub organization settings
   - Navigate to Actions → Runners
   - You should see your runner listed as "Idle" or "Active"

5. **View logs**:
   ```bash
   docker compose logs -f
   ```

6. **Stop the runner** (automatically deregisters):
   ```bash
   docker compose down
   ```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GITHUB_ORGANIZATION` | Yes | - | GitHub organization name (without URL prefix) |
| `GITHUB_ACCESS_TOKEN` | Yes | - | Personal access token with `repo` and `admin:org` scopes |
| `RUNNER_NAME` | Yes | - | Unique identifier for this runner instance |
| `RUNNER_LABELS` | No | `self-hosted,Linux,X64` | Comma-separated labels for workflow targeting |

### Creating a GitHub Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token with these scopes:
   - `repo` (Full control of private repositories)
   - `admin:org` (Full control of orgs and teams, read and write org projects)
3. Copy the token and add it to your `.env` file

## How It Works

1. **Startup**: The entrypoint script validates environment variables and fetches a registration token from GitHub API
2. **Registration**: Configures and registers the runner with your GitHub organization
3. **Running**: Starts the runner process and begins accepting workflow jobs
4. **Shutdown**: When a termination signal is received, the cleanup function deregisters the runner from GitHub Actions before exiting

This prevents orphaned runners from accumulating in your GitHub organization settings.

## Installed Tools

The runner image comes pre-installed with tools commonly needed for CI/CD workflows:

- **Version Control**: git
- **Shell & Utilities**: bash, curl, wget, jq, less
- **Compression**: tar, gzip, unzip
- **Programming Languages**:
  - Node.js
  - Python 3 (with pip and venv support)
- **Build Tools**: build-essential, libssl-dev, libffi-dev
- **SSH**: openssh-client, sshpass
- **Configuration Management**:
  - Ansible (ansible-core 2.17.7)
  - Ansible collections (ansible 10.7.0)
- **Data Processing**: yq (YAML/JSON processor)

These tools enable the runner to handle a wide variety of workflow tasks without requiring additional setup steps.

## Troubleshooting

### Runner fails to start

- Verify your `GITHUB_ACCESS_TOKEN` has the correct scopes
- Check that `GITHUB_ORGANIZATION` matches your org name exactly (case-sensitive)
- Review logs: `docker compose logs`

### Runner not appearing in GitHub

- Ensure your access token has `admin:org` scope
- Verify the organization name is correct (not the URL, just the org name)
- Check that the token hasn't expired

### Runner stays in "Offline" state

- The container may have stopped. Check: `docker compose ps`
- View logs for errors: `docker compose logs`

## Security Considerations

- **Never commit `.env` file**: The `.env` file contains sensitive tokens
- **Token Permissions**: Use a token with minimal required scopes
- **Runner as Root**: This runner runs as root for simplicity. For production use, consider implementing a non-root user
- **Network Isolation**: The runner has no persistent storage but can access the network to communicate with GitHub

## License

This project is provided as-is for use with GitHub Actions self-hosted runners.
