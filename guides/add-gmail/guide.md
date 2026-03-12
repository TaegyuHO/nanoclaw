---
name: add-gmail
description: Add Gmail integration — as a tool (read/send emails) or full channel (inbox polling triggers agent).
tags: [channel, gmail, email, google]
requires:
  tools: [git, npm]
  services: [gcp-oauth]
---

# Add Gmail Integration

## Overview

Adds Gmail to NanoClaw in one of two modes:
- **Tool mode** — Agent gets Gmail tools (read, send, search, draft) but doesn't monitor inbox
- **Channel mode** — Agent listens on Gmail, incoming emails trigger responses automatically

## Prerequisites

- A Google account
- Google Cloud Platform project (free tier sufficient)
- NanoClaw installed and running

## GCP OAuth Setup

1. Open [console.cloud.google.com](https://console.cloud.google.com) → create/select project
2. **APIs & Services → Library** → search "Gmail API" → **Enable**
3. **APIs & Services → Credentials** → **+ CREATE CREDENTIALS → OAuth client ID**
   - If prompted for consent screen: choose "External", fill in app name and email
   - Application type: **Desktop app**
4. **Download JSON** → save as `gcp-oauth.keys.json`

Place credentials:

```bash
mkdir -p ~/.gmail-mcp
cp /path/to/gcp-oauth.keys.json ~/.gmail-mcp/gcp-oauth.keys.json
```

Authorize:

```bash
npx -y @gongrzhe/server-gmail-autoauth-mcp auth
```

A browser opens — sign in and grant access. If "app isn't verified" warning appears, click Advanced → Go to app (unsafe). This is normal for personal OAuth apps.

Verify: `ls ~/.gmail-mcp/credentials.json`

## Code Installation

```bash
git remote add gmail https://github.com/qwibitai/nanoclaw-gmail.git
git fetch gmail main
git merge gmail/main || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/channels/gmail.ts` — GmailChannel class (channel mode)
- Gmail MCP server config in `container/agent-runner/src/index.ts`
- `~/.gmail-mcp` mount in `src/container-runner.ts`
- `googleapis` dependency

Validate:

```bash
npm install && npm run build
npx vitest run src/channels/gmail.test.ts
```

## Post-Installation

Clear stale agent-runner copies and rebuild container:

```bash
rm -r data/sessions/*/agent-runner-src 2>/dev/null || true
cd container && ./build.sh && cd ..
npm run build
```

Restart service:

```bash
# macOS: launchctl kickstart -k gui/$(id -u)/com.nanoclaw
# Linux: systemctl --user restart nanoclaw
```

## Channel Mode Setup

For channel mode (inbox polling), add email handling instructions to `groups/main/CLAUDE.md`:

```markdown
## Email Notifications

When you receive an email notification (messages starting with `[Email from ...`), inform the user but do NOT reply unless specifically asked. Use Gmail tools only when explicitly requested.
```

Default filter: unread Primary inbox emails (`is:unread category:primary`). Promotions, Social, Updates, Forums are excluded.

## Verification

Test tool access:
> Send in main channel: "check my recent emails" or "list my Gmail labels"

Test channel mode:
> Send yourself a test email. Agent should pick it up within a minute.

Monitor: `tail -f logs/nanoclaw.log | grep -iE "(gmail|email)"`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Gmail not responding | Test: `npx -y @gongrzhe/server-gmail-autoauth-mcp` |
| OAuth token expired | `rm ~/.gmail-mcp/credentials.json` and re-authorize |
| Container can't access Gmail | Verify `~/.gmail-mcp` mount in `src/container-runner.ts` |
| Emails not detected (channel mode) | Check polling logs. Default query: `is:unread category:primary` |

## Removal

**Tool-only:** Remove `~/.gmail-mcp` mount and Gmail MCP server config, rebuild container.

**Channel mode:** Also delete `src/channels/gmail.ts`, `src/channels/gmail.test.ts`, remove import from `src/channels/index.ts`, `npm uninstall googleapis`.
