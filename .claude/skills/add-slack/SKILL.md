---
name: add-slack
description: Add Slack as a channel. Can replace WhatsApp entirely or run alongside it. Uses Socket Mode (no public URL needed).
guide: ../../guides/add-slack/guide.md
---

# Add Slack Channel

Read the guide at `guides/add-slack/guide.md` for domain knowledge (Slack app creation, Socket Mode, OAuth scopes, troubleshooting). This skill orchestrates those steps via Claude Code tools.

## Phase 1: Pre-flight

Check if `src/channels/slack.ts` exists. If yes, skip to Phase 3.

`AskUserQuestion`: Do you have a Slack app configured? If yes, collect Bot Token and App Token.

## Phase 2: Apply Code Changes

Merge channel code per the guide. Validate with build + tests.

## Phase 3: Setup

If user needs a Slack app, share `SLACK_SETUP.md` reference and walk through the steps (see guide for quick summary).

Collect both tokens. Add to `.env`:
```
SLACK_BOT_TOKEN=xoxb-...
SLACK_APP_TOKEN=xapp-...
```

Sync: `mkdir -p data/env && cp .env data/env/env`. Build and restart.

## Phase 4: Registration

Help user get channel ID (see guide for methods). Register with `npx tsx setup/index.ts --step register`. Remind to add bot to the channel.

## Phase 5: Verify

Test message. `tail -f logs/nanoclaw.log`.
