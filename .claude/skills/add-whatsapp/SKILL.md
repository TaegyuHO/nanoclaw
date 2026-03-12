---
name: add-whatsapp
description: Add WhatsApp as a channel. Can replace other channels entirely or run alongside them. Uses QR code or pairing code for authentication.
guide: ../../guides/add-whatsapp/guide.md
---

# Add WhatsApp Channel

Read the guide at `guides/add-whatsapp/guide.md` for domain knowledge (auth methods, JID formats, troubleshooting). This skill orchestrates those steps via Claude Code tools.

## Phase 1: Pre-flight

Check `store/auth/creds.json` — if exists, skip to Phase 4. Detect headless env.

`AskUserQuestion` (adapt options based on environment): How do you want to authenticate WhatsApp?
- QR code in browser (Recommended for desktop)
- Pairing code (for headless)
- QR code in terminal

If pairing code: `AskUserQuestion`: What is your phone number? (country code without +)

## Phase 2: Apply Code Changes

Check if `src/channels/whatsapp.ts` exists. If not, merge channel code per the guide. Validate with build + tests.

## Phase 3: Authentication

Clean previous state: `rm -rf store/auth/`

Run auth per chosen method (see guide for exact commands). For pairing code: run in background, poll `store/pairing-code.txt`, display code immediately — it expires in ~60s.

Verify: `test -f store/auth/creds.json`

## Phase 4: Registration

`AskUserQuestion`: Shared or dedicated number?
`AskUserQuestion`: Trigger word? (default @Andy)
`AskUserQuestion`: Assistant name?
`AskUserQuestion`: Where to chat? (self-chat, solo group, existing group)

Get JID per choice (see guide). Register with `npx tsx setup/index.ts --step register`.

## Phase 5: Verify

Build, restart, test. `tail -f logs/nanoclaw.log`.
