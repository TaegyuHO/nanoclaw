---
name: add-telegram
description: Add Telegram bot as a messaging channel for NanoClaw.
tags: [channel, telegram, messaging]
requires:
  tools: [git, npm]
  services: [telegram-bot]
---

# Add Telegram Channel

## Overview

Adds Telegram as a messaging channel using the grammy library. The bot connects via Telegram's Bot API. Can run alongside other channels or as the sole channel.

## Prerequisites

- A Telegram account
- NanoClaw installed and running

## Creating a Telegram Bot

1. Open Telegram and search for `@BotFather`
2. Send `/newbot` and follow prompts:
   - Bot name: Something friendly (e.g., "Andy Assistant")
   - Bot username: Must end with "bot" (e.g., "andy_ai_bot")
3. Copy the bot token (looks like `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11`)

## Code Installation

The channel code lives in a separate repository and is merged via git:

```bash
git remote add telegram https://github.com/qwibitai/nanoclaw-telegram.git
git fetch telegram main
git merge telegram/main || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/channels/telegram.ts` — TelegramChannel class with self-registration
- `src/channels/telegram.test.ts` — Unit tests
- `grammy` npm dependency

Validate:

```bash
npm install && npm run build
npx vitest run src/channels/telegram.test.ts
```

## Configuration

Add to `.env`:

```bash
TELEGRAM_BOT_TOKEN=<your-token>
```

Sync to container:

```bash
mkdir -p data/env && cp .env data/env/env
```

## Group Privacy (Important for Group Chats)

By default, Telegram bots only see @mentions and commands in groups. To let the bot see all messages:

1. Open `@BotFather` → `/mybots` → select your bot
2. **Bot Settings** → **Group Privacy** → **Turn off**

This is optional if you only want trigger-based responses.

## Registration

Get the chat ID by sending `/chatid` to your bot (or in a group after adding the bot).

JID format: `tg:123456789` (DM) or `tg:-1001234567890` (group)

Register as main:

```bash
npx tsx setup/index.ts --step register -- --jid "tg:<chat-id>" --name "<name>" --folder "telegram_main" --trigger "@Andy" --channel telegram --no-trigger-required --is-main
```

Register as additional:

```bash
npx tsx setup/index.ts --step register -- --jid "tg:<chat-id>" --name "<name>" --folder "telegram_<name>" --trigger "@Andy" --channel telegram
```

## Verification

Build and restart:

```bash
npm run build
# macOS: launchctl kickstart -k gui/$(id -u)/com.nanoclaw
# Linux: systemctl --user restart nanoclaw
```

Send a test message. Monitor: `tail -f logs/nanoclaw.log`

## Agent Swarms

After setup, Agent Swarm support can be added so each subagent appears as a different bot in the Telegram group. See the add-telegram-swarm guide.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Bot not responding | Check `TELEGRAM_BOT_TOKEN` in `.env` and `data/env/env`. Verify registration: `sqlite3 store/messages.db "SELECT * FROM registered_groups WHERE jid LIKE 'tg:%'"` |
| Only responds to @mentions in groups | Group Privacy is on. Disable via BotFather, then remove and re-add bot to group. |
| Can't get chat ID | Verify token: `curl -s "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/getMe"` |

## Removal

1. Delete `src/channels/telegram.ts` and `src/channels/telegram.test.ts`
2. Remove `import './telegram.js'` from `src/channels/index.ts`
3. Remove `TELEGRAM_BOT_TOKEN` from `.env`
4. Remove registrations: `sqlite3 store/messages.db "DELETE FROM registered_groups WHERE jid LIKE 'tg:%'"`
5. `npm uninstall grammy && npm run build`
