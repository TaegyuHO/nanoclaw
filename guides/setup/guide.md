---
name: setup
description: Install dependencies, configure container runtime, authenticate Claude, set up channels, and start the NanoClaw service.
tags: [core, setup, installation]
requires:
  tools: [git, node]
  runtime: "node >= 20"
---

# NanoClaw Setup Guide

## Overview

NanoClaw requires: Node.js 20+, a container runtime (Docker or Apple Container), and Claude authentication. Setup proceeds in order: bootstrap → container → auth → channels → service.

## Prerequisites

- Git repository (cloned from `TaegyuHO/nanoclaw`)
- Node.js 20 or later
- Docker Desktop (all platforms) or Apple Container (macOS only)
- Claude subscription (Pro/Max) or Anthropic API key

## Steps

### 1. Git & Fork Configuration

Verify remotes:

```bash
git remote -v
```

Expected: `origin` → `TaegyuHO/nanoclaw.git`, `upstream` → `qwibitai/nanoclaw.git` (optional).

**If no `upstream` remote (for pulling upstream updates):**

```bash
git remote add upstream https://github.com/qwibitai/nanoclaw.git
```

### 2. Bootstrap (Node.js + Dependencies)

```bash
bash setup.sh
```

This checks Node.js version, installs npm dependencies, and verifies native modules (better-sqlite3).

**If Node.js is missing or too old:**

| Platform | Install Command |
|----------|----------------|
| macOS | `brew install node@22` or install via [nvm](https://github.com/nvm-sh/nvm) |
| Linux | `curl -fsSL https://deb.nodesource.com/setup_22.x \| sudo -E bash - && sudo apt-get install -y nodejs` |

**If native module build fails:** Install build tools (`xcode-select --install` on macOS, `build-essential` on Linux), then re-run `bash setup.sh`.

### 3. Container Runtime

NanoClaw runs agents inside containers for isolation.

**Choosing a runtime:**

| Platform | Options |
|----------|---------|
| Linux | Docker (only option) |
| macOS | Docker or Apple Container (native) |

**Docker installation:**

| Platform | Install |
|----------|---------|
| macOS | `brew install --cask docker` then `open -a Docker` |
| Linux | `curl -fsSL https://get.docker.com \| sh && sudo usermod -aG docker $USER` |

**If Docker is installed but not running:**

```bash
# macOS
open -a Docker

# Linux
sudo systemctl start docker
```

**Build and test the container image:**

```bash
npx tsx setup/index.ts --step container -- --runtime docker
```

If build fails due to cache: `docker builder prune -f` and retry.

### 4. Claude Authentication

Add one of these to `.env`:

```bash
# Claude subscription (Pro/Max)
CLAUDE_CODE_OAUTH_TOKEN=<token>

# Anthropic API key (pay-per-use)
ANTHROPIC_API_KEY=<key>
```

For subscription: run `claude setup-token` in a separate terminal to get the token.

### 5. Channel Setup

Each channel is installed separately via its own guide. Enable one or more:

- [WhatsApp](../add-whatsapp/guide.md) — QR code or pairing code authentication
- [Telegram](../add-telegram/guide.md) — Bot token from @BotFather
- [Slack](../add-slack/guide.md) — Slack app with Socket Mode
- [Discord](../add-discord/guide.md) — Discord bot token

After installing channels, rebuild:

```bash
npm install && npm run build
```

### 6. Mount Allowlist

Configure which host directories agents can access:

```bash
# No external access (default)
npx tsx setup/index.ts --step mounts -- --empty

# Custom mounts
npx tsx setup/index.ts --step mounts -- --json '{"allowedRoots":["/path/to/dir"],"blockedPatterns":[],"nonMainReadOnly":true}'
```

### 7. Start Service

```bash
npx tsx setup/index.ts --step service
```

**macOS (launchd):**

```bash
launchctl load ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl kickstart -k gui/$(id -u)/com.nanoclaw    # restart
launchctl unload ~/Library/LaunchAgents/com.nanoclaw.plist  # stop
```

**Linux (systemd):**

```bash
systemctl --user start nanoclaw
systemctl --user restart nanoclaw
systemctl --user stop nanoclaw
```

**WSL without systemd:** Enable systemd (`echo -e "[boot]\nsystemd=true" | sudo tee /etc/wsl.conf`, restart WSL) or use the generated `start-nanoclaw.sh` wrapper.

**Docker group issue on Linux:** If the service can't reach the Docker socket:

```bash
sudo setfacl -m u:$(whoami):rw /var/run/docker.sock
```

### 8. Verify

```bash
npx tsx setup/index.ts --step verify
```

Check each status field: SERVICE, CREDENTIALS, CHANNEL_AUTH, REGISTERED_GROUPS, MOUNT_ALLOWLIST. Fix any failures by re-running the relevant step above.

Test by sending a message in your registered chat, then monitor:

```bash
tail -f logs/nanoclaw.log
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Service not starting | Check `logs/nanoclaw.error.log`. Common: wrong Node path, missing `.env`, missing channel credentials. |
| Container agent fails ("exited with code 1") | Ensure container runtime is running. Check `groups/main/logs/container-*.log`. |
| No response to messages | Check trigger pattern (main channel doesn't need prefix). Run verify step. Check `logs/nanoclaw.log`. |
| Channel not connecting | Verify credentials in `.env`. Restart service after `.env` changes. |
