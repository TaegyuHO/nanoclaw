---
name: add-reactions
description: Add WhatsApp emoji reaction support — receive, send, store, and search reactions.
tags: [feature, reactions, emoji, whatsapp]
requires:
  tools: [git, npm]
  prerequisites: [whatsapp-channel]
---

# Add Reactions

## Overview

Adds emoji reaction support to WhatsApp: receive and store reactions in SQLite, send reactions from the agent via MCP tool, and query reaction history.

## Prerequisites

- WhatsApp channel installed

## Code Installation

```bash
git remote add whatsapp https://github.com/qwibitai/nanoclaw-whatsapp.git  # if not already added
git fetch whatsapp skill/reactions
git merge whatsapp/skill/reactions || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `scripts/migrate-reactions.ts` — Database migration
- `src/status-tracker.ts` — Emoji state machine
- `container/skills/reactions/SKILL.md` — Agent-facing `react_to_message` MCP tool docs

Run migration:

```bash
npx tsx scripts/migrate-reactions.ts
```

Validate: `npm test && npm run build`

Restart service.

## Verification

1. Send a message, react to it with an emoji on WhatsApp
2. Check DB: `sqlite3 store/messages.db "SELECT * FROM reactions ORDER BY timestamp DESC LIMIT 5;"`
3. Ask agent to react to a message via the `react_to_message` tool

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Reactions not in DB | Check logs for errors. Verify chat is registered. |
| Migration fails | If "table already exists", migration already ran. |
| Agent can't send reactions | Check IPC logs for authorization errors — agent can only react in its own group. |
