---
name: add-telegram
description: Add Telegram as a channel. Can replace WhatsApp entirely or run alongside it. Also configurable as a control-only channel (triggers actions) or passive channel (receives notifications only).
guide: ../../guides/add-telegram/guide.md
---

# Add Telegram Channel

Read the guide at `guides/add-telegram/guide.md` for domain knowledge (BotFather setup, group privacy, chat ID formats, troubleshooting). This skill orchestrates those steps via Claude Code tools.

## Phase 1: Pre-flight

Check if `src/channels/telegram.ts` exists. If yes, skip to Phase 3.

`AskUserQuestion`: Do you have a Telegram bot token, or do you need to create one?

## Phase 2: Apply Code Changes

Merge the channel code per the guide (git remote, fetch, merge). Handle package-lock conflicts. Run `npm install && npm run build && npx vitest run src/channels/telegram.test.ts`.

## Phase 3: Setup

If user needs a bot token, walk them through BotFather (see guide). Collect token.

Add `TELEGRAM_BOT_TOKEN=<token>` to `.env`. Sync: `mkdir -p data/env && cp .env data/env/env`.

Remind about Group Privacy if they plan to use groups (see guide).

Build and restart.

## Phase 4: Registration

Tell user to send `/chatid` to the bot. Collect chat ID. Register using `npx tsx setup/index.ts --step register` with appropriate flags (see guide for main vs additional).

## Phase 5: Verify

Tell user to send a test message. Check `tail -f logs/nanoclaw.log`.

`AskUserQuestion`: Would you like to add Agent Swarm support? If yes, invoke `/add-telegram-swarm`.
