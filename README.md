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

4. **Open the web UI**

   Go to http://127.0.0.1:18789/ and enter the WebSocket URL (`ws://127.0.0.1:18789`) and gateway token (from `.env` or `openclaw-config/openclaw.json` under `gateway.auth.token`).

5. **Approve browser pairing** (required once per browser)

   After you click **Connect**, the dashboard may show **pairing required**. The gateway must approve your browser as a device. With the gateway running:

   ```bash
   docker compose --profile cli run --rm openclaw-cli devices list
   docker compose --profile cli run --rm openclaw-cli devices approve --latest
   ```

   Then click **Connect** again in the browser.

   Optional: run interactive onboarding (providers, channels) if you prefer a wizard:

   ```bash
   docker compose --profile cli run --rm openclaw-cli onboard
   ```

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
docker compose --profile cli run --rm openclaw-cli <command>

# Add a Telegram channel
docker compose --profile cli run --rm openclaw-cli channels add --channel telegram --token "<bot-token>"

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

## Troubleshooting

### "pairing required" in the Control UI

OpenClaw treats the dashboard as a **paired device**, not only token auth. After connecting once from the browser, approve the pending request from the CLI (see step 5 in Quick Start). Use `devices list` if you need the request ID, or `devices approve --latest` for the most recent pending request.

### Tokenized dashboard URL (alternative)

```bash
docker compose --profile cli run --rm openclaw-cli dashboard --no-open
```

Paste the printed URL in the browser if you prefer the wizard-generated link.
