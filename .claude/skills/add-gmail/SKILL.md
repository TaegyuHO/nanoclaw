---
name: add-gmail
description: Add Gmail integration to NanoClaw. Can be configured as a tool (agent reads/sends emails when triggered from WhatsApp) or as a full channel (emails can trigger the agent, schedule tasks, and receive replies). Guides through GCP OAuth setup and implements the integration.
guide: ../../guides/add-gmail/guide.md
---

# Add Gmail Integration

Read the guide at `guides/add-gmail/guide.md` for domain knowledge (GCP OAuth, tool vs channel mode, troubleshooting). This skill orchestrates those steps via Claude Code tools.

## Phase 1: Pre-flight

Check if `src/channels/gmail.ts` exists. If yes, skip to Phase 3.

`AskUserQuestion`: Should incoming emails be able to trigger the agent?
- Yes — Full channel mode
- No — Tool-only mode

## Phase 2: Apply Code Changes

Merge channel code per the guide. If channel mode, add email handling instructions to `groups/main/CLAUDE.md`. Validate with build + tests.

## Phase 3: Setup

Check `~/.gmail-mcp/credentials.json`. If exists, skip to build.

Walk user through GCP OAuth setup (see guide). Collect `gcp-oauth.keys.json` path or contents. Copy to `~/.gmail-mcp/`.

Run authorization: `npx -y @gongrzhe/server-gmail-autoauth-mcp auth`

Clear stale agent-runner copies, rebuild container, compile, restart.

## Phase 4: Verify

Test tool access: "check my recent emails". For channel mode: send a test email, monitor logs.

`AskUserQuestion`: Want to customize email filters? (default: Primary inbox only)
