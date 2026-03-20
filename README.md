# OpenClaw Docker Sandbox

Run OpenClaw AI assistant in Docker so it cannot access your host machine beyond the explicitly mounted directories.

## Prerequisites

- Docker Engine 24+ (27+ recommended)
- Docker Compose v2.20+ (v2.30+ recommended)

## Quick Start

1. **Configure API keys**

   ```bash
   cp .env.example .env
   ```

   Edit `.env` and add your OpenAI and/or Anthropic API keys.

2. **Create the mount directories**

   ```bash
   mkdir -p openclaw-config workspace
   ```

3. **Start the gateway**

   ```bash
   docker compose up -d
   ```

4. **Run onboarding** (first time only)

   ```bash
   docker compose run --rm --profile cli openclaw-cli onboard
   ```

   This generates a gateway token and writes it to your config. Copy the token.

5. **Open the web UI**

   Go to http://127.0.0.1:18789/ and paste the gateway token into Settings.

## Common Commands

```bash
# Stop
docker compose down

# Restart
docker compose restart

# View logs
docker compose logs -f openclaw-gateway

# Open a shell in the gateway container
docker compose exec openclaw-gateway bash

# Run any CLI command
docker compose run --rm --profile cli openclaw-cli <command>

# Add a Telegram channel
docker compose run --rm --profile cli openclaw-cli channels add --channel telegram --token "<bot-token>"

# Check health
curl -fsS http://127.0.0.1:18789/healthz
```

## What's Mounted

| Host Path | Container Path | Purpose |
|---|---|---|
| `./openclaw-config/` | `/home/node/.openclaw` | Config, API keys, agent memory |
| `./workspace/` | `/home/node/.openclaw/workspace` | Agent's working directory |

The container has **no access** to any other host directory (home, ssh, aws, etc.). Ports are bound to `127.0.0.1` only -- not exposed to your LAN.

## Security Model

- **Host isolation**: Only the two directories above are visible to the container. Your host filesystem is completely hidden.
- **Unrestricted inside the container**: The agent can install packages, run commands, and write files freely within its sandbox.
- **Localhost-only ports**: The web UI (18789) and bridge (18790) are bound to `127.0.0.1`, preventing network exposure.
