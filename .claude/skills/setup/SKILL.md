---
name: setup
description: Run initial NanoClaw setup. Use when user wants to install dependencies, authenticate messaging channels, register their main channel, or start the background services. Triggers on "setup", "install", "configure nanoclaw", or first-time setup requests.
guide: ../../guides/setup/guide.md
---

# NanoClaw Setup

Read the guide at `guides/setup/guide.md` for domain knowledge (installation steps, platform specifics, troubleshooting). This skill orchestrates those steps via Claude Code tools.

**Principle:** When something is broken or missing, fix it. Don't tell the user to go fix it themselves unless it genuinely requires their manual action (e.g. authenticating a channel, pasting a secret token).

**UX Note:** Use `AskUserQuestion` for all user-facing questions.

## 0. Git & Fork Setup

Run `git remote -v`. Handle cases per the guide (Case A/B/C). Use `AskUserQuestion` if the user needs to fork.

## 1. Bootstrap

Run `bash setup.sh` and parse the status block.
- NODE_OK=false → `AskUserQuestion: Install Node.js 22?` Then install per platform (see guide).
- DEPS_OK=false → Read `logs/setup.log`, delete `node_modules`, re-run.
- NATIVE_OK=false → Install build tools, retry.
- Record PLATFORM and IS_WSL.

## 2. Check Environment

Run `npx tsx setup/index.ts --step environment` and parse the status block.
- Record HAS_AUTH, HAS_REGISTERED_GROUPS, APPLE_CONTAINER, DOCKER.

## 3. Container Runtime

### 3a. Choose runtime
- PLATFORM=linux → Docker
- PLATFORM=macos + APPLE_CONTAINER=installed → `AskUserQuestion: Docker or Apple Container?` If Apple Container → run `/convert-to-apple-container`.
- PLATFORM=macos + APPLE_CONTAINER=not_found → Docker

### 3a-docker. Install Docker
- DOCKER=running → continue
- DOCKER=installed_not_running → start it (see guide). Wait 15s, re-check `docker info`.
- DOCKER=not_found → `AskUserQuestion: Install Docker?` Then install per platform.

### 3b. Apple Container conversion gate
If Apple Container chosen, check: `grep -q "CONTAINER_RUNTIME_BIN = 'container'" src/container-runtime.ts`. If NEEDS_CONVERSION → run `/convert-to-apple-container`.

### 3c. Build and test
Run `npx tsx setup/index.ts --step container -- --runtime <chosen>`. Handle BUILD_OK/TEST_OK failures per guide.

## 4. Claude Authentication

If HAS_ENV=true, check `.env` for existing credentials. `AskUserQuestion: Claude subscription or API key?`
- Subscription → tell user to run `claude setup-token`, add token to `.env`.
- API key → tell user to add key to `.env`.

## 5. Set Up Channels

`AskUserQuestion (multiSelect): Which channels?` — WhatsApp, Telegram, Slack, Discord.

Delegate to each channel's skill: `/add-whatsapp`, `/add-telegram`, `/add-slack`, `/add-discord`.

After all complete: `npm install && npm run build`.

## 6. Mount Allowlist

`AskUserQuestion: Agent access to external directories?`
- No: `npx tsx setup/index.ts --step mounts -- --empty`
- Yes: Collect paths. `npx tsx setup/index.ts --step mounts -- --json '...'`

## 7. Start Service

If service already running, unload first (see guide for platform commands).
Run `npx tsx setup/index.ts --step service`. Handle FALLBACK, DOCKER_GROUP_STALE, SERVICE_LOADED failures per guide.

## 8. Verify

Run `npx tsx setup/index.ts --step verify`. Fix each failure by re-running the relevant step.
Tell user to test: send a message. Show: `tail -f logs/nanoclaw.log`.
