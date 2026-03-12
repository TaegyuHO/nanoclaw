---
name: add-discord
description: Add Discord bot channel integration to NanoClaw.
guide: ../../guides/add-discord/guide.md
---

# Add Discord Channel

Read the guide at `guides/add-discord/guide.md` for domain knowledge (bot creation, intents, channel ID, troubleshooting). This skill orchestrates those steps via Claude Code tools.

## Phase 1: Pre-flight

Check if `src/channels/discord.ts` exists. If yes, skip to Phase 3.

`AskUserQuestion`: Do you have a Discord bot token, or do you need to create one?

## Phase 2: Apply Code Changes

Merge channel code per the guide. Validate with build + tests.

## Phase 3: Setup

If user needs a bot, walk through Developer Portal steps (see guide). Collect token.

Add `DISCORD_BOT_TOKEN=<token>` to `.env`. Sync: `mkdir -p data/env && cp .env data/env/env`. Build and restart.

## Phase 4: Registration

Help user enable Developer Mode and copy channel ID (see guide). Register with `npx tsx setup/index.ts --step register`.

## Phase 5: Verify

Test message. `tail -f logs/nanoclaw.log`.
