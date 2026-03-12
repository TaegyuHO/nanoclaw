---
name: add-slack
description: Add Slack as a messaging channel using Socket Mode (no public URL needed).
tags: [channel, slack, messaging]
requires:
  tools: [git, npm]
  services: [slack-app]
---

# Add Slack Channel

## Overview

Adds Slack as a messaging channel using the @slack/bolt library with Socket Mode. No public URL or webhook endpoint required.

## Prerequisites

- A Slack workspace where you can install apps
- NanoClaw installed and running

## Creating a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps) → **Create New App** → **From scratch**
2. **Socket Mode**: Enable and generate an App-Level Token (`xapp-...`)
3. **Event Subscriptions**: Subscribe to bot events: `message.channels`, `message.groups`, `message.im`
4. **OAuth & Permissions**: Add scopes: `chat:write`, `channels:history`, `groups:history`, `im:history`, `channels:read`, `groups:read`, `users:read`
5. **Install to Workspace**: Copy the Bot Token (`xoxb-...`)

See `SLACK_SETUP.md` in the skill directory for detailed step-by-step with screenshots guidance.

## Code Installation

```bash
git remote add slack https://github.com/qwibitai/nanoclaw-slack.git
git fetch slack main
git merge slack/main || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/channels/slack.ts` — SlackChannel class with self-registration
- `@slack/bolt` dependency

Validate:

```bash
npm install && npm run build
npx vitest run src/channels/slack.test.ts
```

## Configuration

Add to `.env`:

```bash
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-token
```

Sync to container:

```bash
mkdir -p data/env && cp .env data/env/env
```

## Registration

Get the channel ID:
- Browser URL: `https://app.slack.com/client/T.../C0123456789` → the `C...` part
- Right-click channel → Copy link → last path segment
- API: `curl -s -H "Authorization: Bearer $SLACK_BOT_TOKEN" "https://slack.com/api/conversations.list" | jq '.channels[] | {id, name}'`

JID format: `slack:C0123456789`

Register:

```bash
npx tsx setup/index.ts --step register -- --jid "slack:<channel-id>" --name "<name>" --folder "slack_main" --trigger "@Andy" --channel slack --no-trigger-required --is-main
```

**Important:** Add the bot to each channel you want it to monitor (right-click channel → View channel details → Integrations → Add apps).

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
| Bot not responding | Check both tokens in `.env` and `data/env/env`. Verify registration. |
| Not receiving messages | Verify Socket Mode enabled, bot events subscribed, bot added to channel. |
| "missing_scope" errors | Add scope in OAuth & Permissions, **reinstall app**, update new bot token. |

## Known Limitations

- Threaded replies are flattened — responses go to channel, not back into thread
- No typing indicator (Slack Bot API limitation)
- Long messages split at 4000-char boundary
- No file/image handling — text only
