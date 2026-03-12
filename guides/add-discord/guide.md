---
name: add-discord
description: Add Discord bot as a messaging channel.
tags: [channel, discord, messaging]
requires:
  tools: [git, npm]
  services: [discord-bot]
---

# Add Discord Channel

## Overview

Adds Discord as a messaging channel using the discord.js library. Supports text messages, attachment descriptions, reply context, @mention translation, and typing indicators.

## Prerequisites

- A Discord server where you have admin/manage permissions
- NanoClaw installed and running

## Creating a Discord Bot

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) → **New Application**
2. **Bot** tab → **Reset Token** → copy token immediately (shown once)
3. Under **Privileged Gateway Intents**, enable:
   - **Message Content Intent** (required to read messages)
   - **Server Members Intent** (optional, for display names)
4. **OAuth2** → **URL Generator**:
   - Scopes: `bot`
   - Bot Permissions: `Send Messages`, `Read Message History`, `View Channels`
   - Copy generated URL → open in browser to invite bot to server

## Code Installation

```bash
git remote add discord https://github.com/qwibitai/nanoclaw-discord.git
git fetch discord main
git merge discord/main || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/channels/discord.ts` — DiscordChannel class with self-registration
- `discord.js` dependency

Validate:

```bash
npm install && npm run build
npx vitest run src/channels/discord.test.ts
```

## Configuration

Add to `.env`:

```bash
DISCORD_BOT_TOKEN=<your-token>
```

Sync to container:

```bash
mkdir -p data/env && cp .env data/env/env
```

## Registration

Get the channel ID:
1. Discord → User Settings → Advanced → Enable **Developer Mode**
2. Right-click text channel → **Copy Channel ID**

JID format: `dc:1234567890123456`

Register:

```bash
npx tsx setup/index.ts --step register -- --jid "dc:<channel-id>" --name "<server> #<channel>" --folder "discord_main" --trigger "@Andy" --channel discord --no-trigger-required --is-main
```

## Verification

```bash
npm run build
# macOS: launchctl kickstart -k gui/$(id -u)/com.nanoclaw
# Linux: systemctl --user restart nanoclaw
```

Send a test message. Monitor: `tail -f logs/nanoclaw.log`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot not responding | Check `DISCORD_BOT_TOKEN` in `.env` and `data/env/env`. Verify bot invited to server. |
| Can't read messages | Enable **Message Content Intent** in Developer Portal → Bot tab. |
| Can't copy channel ID | Enable **Developer Mode**: User Settings → Advanced. |
