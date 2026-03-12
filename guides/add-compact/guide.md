---
name: add-compact
description: Add /compact command for manual context compaction in long sessions.
tags: [feature, session, compact]
requires:
  tools: [git, npm, docker]
---

# Add /compact Command

## Overview

Adds a `/compact` session command that compacts conversation history to fight context rot in long sessions. Uses the Claude Agent SDK's built-in `/compact` slash command. The session continues with summarized context — not a destructive reset.

## Security

- Main-group or trusted/admin sender only
- Device owner (`is_from_me`) can compact from any group
- Non-admin users in non-main groups are denied

## Code Installation

```bash
git fetch upstream skill/compact
git merge upstream/skill/compact
```

This adds:
- `src/session-commands.ts` — Command parsing and authorization
- Session command interception in `src/index.ts`
- Slash command handling in agent-runner

Validate: `npm test && npm run build`

Rebuild container: `./container/build.sh`

Restart service.

## Usage

Send `/compact` in the main group (or `@Andy /compact` in non-main groups as admin).

The agent acknowledges compaction and continues the conversation with summarized context. A transcript archive is saved to `groups/{folder}/conversations/`.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Session commands require admin access" | Only device owner or main-group senders can use `/compact`. |
| No compact_boundary in logs | SDK may not emit in all versions. Compaction may still have succeeded. |
