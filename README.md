# OpenClaw

OpenClaw is an open-source AI agent platform that allows you to run and manage AI assistants locally or in the cloud.

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running OpenClaw](#running-openclaw)
- [Accessing the Services](#accessing-the-services)
- [Stopping and Cleaning Up](#stopping-and-cleaning-up)
- [Updating OpenClaw](#updating-openclaw)
- [Backup and Restore](#backup-and-restore)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Introduction

This documentation provides instructions for installing and configuring OpenClaw using Docker Compose. OpenClaw consists of several services:
- **Traefik**: Reverse proxy and load balancer
- **OpenClaw Gateway**: Main API gateway for the OpenClaw platform
- **OpenClaw CLI**: Command-line interface for interacting with OpenClaw
- **Cloudflared** (optional): Cloudflare tunnel for exposing services securely

## Prerequisites

Before you begin, ensure you have the following installed:
- [Docker Engine](https://docs.docker.com/engine/install/) (version 20.10 or later)
- [Docker Compose](https://docs.docker.com/compose/install/) (version 2.0 or later)
- Git (for cloning the repository, if applicable)

## Installation

1. **Clone the repository** (if you haven't already):
   ```bash
   git clone https://github.com/openclaw/openclaw.git
   cd openclaw
   ```

2. **Copy the example environment file** and configure it:
   ```bash
   cp .env.example .env   # If .env.example exists, otherwise create .env manually
   ```
   Note: If there's no `.env.example`, you can create a `.env` file based on the variables in the current `.env` file, but be sure to replace sensitive values with your own.

3. **Configure the `.env` file**:
   - Set `DOMAIN_NAME` to your desired domain (e.g., `openclaw.example.com`).
   - Set `ACME_EMAIL` for Let's Encrypt certificates (if using HTTPS).
   - Set `TRAEFIK_DASHBOARD_AUTH` for securing the Traefik dashboard (format: `user:hashed_password`, generate with `htpasswd -nb user password`).
   - Set at least one API key for a model provider (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, etc.).
   - Adjust other variables as needed (see the `.env` file for comments).

## Configuration

### Environment Variables

The `.env` file contains all configuration variables. Here are the most important ones:

| Variable | Description | Example |
|----------|-------------|---------|
| `DOMAIN_NAME` | The domain name for accessing OpenClaw | `openclaw.example.com` |
| `TIMEZONE` | Server timezone | `America/Bogota` |
| `OPENCLAW_TZ` | OpenClaw timezone (usually same as TIMEZONE) | `America/Bogota` |
| `OPENCLAW_GATEWAY_TOKEN` | Secret token for gateway authentication | `your_secret_token_here` |
| `ACME_EMAIL` | Email for Let's Encrypt certificates | `admin@example.com` |
| `TRAEFIK_DASHBOARD_AUTH` | Traefik dashboard basic auth (user:hash) | `admin:$apr1$...` |
| `CF_TUNNEL_TOKEN` | Cloudflare tunnel token (optional) | `eyJ...` |
| `OPENAI_API_KEY` | OpenAI API key (if using OpenAI models) | `sk-...` |
| `ANTHROPIC_API_KEY` | Anthropic API key (if using Claude models) | `sk-ant-...` |
| `GEMINI_API_KEY` | Google Gemini API key | `AI...` |
| `MINIMAX_API_KEY` | MiniMax API key | `sk-cp-...` |

### Volumes and Persistent Data

OpenClaw uses Docker volumes for persistent data:
- `openclaw/data/` - Application data
- `openclaw/workspace/` - Agent workspace
- `openclaw-auth-profile-secrets/` - Authentication secrets
- `traefik/acme.json` - TLS certificates

These directories are bind-mounted from the host, so your data persists between container restarts.

## Running OpenClaw

1. **Start the services**:
   ```bash
   docker compose up -d
   ```
   This will start all defined services in detached mode.

2. **Check the status**:
   ```bash
   docker compose ps
   ```
   You should see all services as "healthy" or "starting".

3. **View logs**:
   ```bash
   docker compose logs -f
   ```
   Or for a specific service:
   ```bash
   docker compose logs -f openclaw-gateway
   ```

## Accessing the Services

Once the services are running:
- **OpenClaw Gateway**: Access via `https://${DOMAIN_NAME}` (or `http://localhost:18789` for local access without Traefik)
- **Traefik Dashboard**: Access via `https://traefik.${DOMAIN_NAME}` (requires `TRAEFIK_DASHBOARD_AUTH` to be set)
- **OpenClaw CLI**: You can run commands using the `openclaw-cli` service:
  ```bash
  docker compose run --rm openclaw-cli your-command-here
  ```

## Stopping and Cleaning Up

- **Stop all services**:
  ```bash
  docker compose down
  ```

- **Stop and remove volumes** (WARNING: This will delete all persistent data!):
  ```bash
  docker compose down -v
  ```

## Updating OpenClaw

To update to the latest version:
```bash
docker compose pull   # Pull latest images
docker compose up -d  # Recreate containers with new images
```

## Backup and Restore

### Backup
To backup your OpenClaw data, simply copy the following directories:
- `openclaw/`
- `openclaw-auth-profile-secrets/`
- `traefik/acme.json`
- `.env` (contains configuration but also secrets - handle securely)

### Restore
1. Stop OpenClaw: `docker compose down`
2. Restore the directories listed above to their original locations.
3. Start OpenClaw: `docker compose up -d`

## Troubleshooting

### Common Issues

1. **Services not starting healthy**:
   - Check logs: `docker compose logs -f <service_name>`
   - Ensure required environment variables are set in `.env`
   - Verify port availability (ports 80, 443, 18789, 18790)

2. **HTTPS/TLS issues**:
   - Check that `ACME_EMAIL` is set correctly in `.env`
   - Ensure ports 80 and 443 are accessible from the internet (for Let's Encrypt validation)
   - Check Traefik logs for certificate issuance errors

3. **Authentication problems**:
   - Verify `OPENCLAW_GATEWAY_TOKEN` in both `.env` and `openclaw/openclaw.json` match
   - Check that `OPENCLAW_GATEWAY_BIND` is set appropriately (`loopback` for local-only, `lan` for network access)

### Getting Help

If you encounter issues not covered here:
- Check the OpenClaw documentation: https://docs.openclaw.dev
- Visit the OpenClaw GitHub repository: https://github.com/openclaw/openclaw
- Open an issue on GitHub with details of your problem

## Additional Resources

- [OpenClaw GitHub Repository](https://github.com/openclaw/openclaw)
- [OpenClaw Documentation](https://docs.openclaw.dev)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

---
*Last updated: May 21, 2026*