---
name: add-whatsapp
description: Add WhatsApp as a messaging channel using QR code or pairing code authentication.
tags: [channel, whatsapp, messaging]
requires:
  tools: [git, npm]
---

# Add WhatsApp Channel

## Overview

Adds WhatsApp as a messaging channel using the baileys library. Authenticates via QR code (browser or terminal) or pairing code. Can use a shared personal number or a dedicated number.

## Prerequisites

- A WhatsApp account
- NanoClaw installed and running

## Authentication Methods

| Method | Best For | Requires |
|--------|----------|----------|
| QR code (browser) | Desktop/WSL | Camera on phone, display on host |
| QR code (terminal) | Any | Camera on phone, terminal display |
| Pairing code | Headless servers | Phone number, no camera needed |

## Code Installation

```bash
git remote add whatsapp https://github.com/qwibitai/nanoclaw-whatsapp.git
git fetch whatsapp main
git merge whatsapp/main || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/channels/whatsapp.ts` — WhatsAppChannel class with self-registration
- `src/whatsapp-auth.ts` — Standalone authentication script
- `@whiskeysockets/baileys`, `qrcode`, `qrcode-terminal` dependencies

Validate:

```bash
npm install && npm run build
npx vitest run src/channels/whatsapp.test.ts
```

## Authentication

Clean previous state if re-authenticating: `rm -rf store/auth/`

**QR code (browser):**

```bash
npx tsx setup/index.ts --step whatsapp-auth -- --method qr-browser
```

Open WhatsApp → Settings → Linked Devices → Link a Device → Scan QR code.

**QR code (terminal):**

```bash
npx tsx setup/index.ts --step whatsapp-auth -- --method qr-terminal
```

**Pairing code:**

```bash
npx tsx setup/index.ts --step whatsapp-auth -- --method pairing-code --phone <phone-number>
```

The code appears in `store/pairing-code.txt`. Enter it in WhatsApp → Settings → Linked Devices → Link a Device → Link with phone number instead. Code expires in ~60 seconds.

Verify: `test -f store/auth/creds.json && echo "Success"`

## Configuration

WhatsApp auto-enables when `store/auth/creds.json` exists. No `.env` variable needed.

Sync env to container:

```bash
mkdir -p data/env && cp .env data/env/env
```

## Registration

**Phone number types:**
- **Shared number** — your personal WhatsApp. Use self-chat or a solo group.
- **Dedicated number** — separate phone/SIM for the assistant.

**Getting the JID:**

Self-chat:
```bash
node -e "const c=JSON.parse(require('fs').readFileSync('store/auth/creds.json','utf-8'));console.log(c.me?.id?.split(':')[0]+'@s.whatsapp.net')"
```

Group:
```bash
npx tsx setup/index.ts --step groups
npx tsx setup/index.ts --step groups --list
```

Register as main:

```bash
npx tsx setup/index.ts --step register \
  --jid "<jid>" --name "<name>" --trigger "@Andy" \
  --folder "whatsapp_main" --channel whatsapp \
  --assistant-name "Andy" --is-main --no-trigger-required
```

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
| QR code expired | Re-run: `rm -rf store/auth/ && npx tsx src/whatsapp-auth.ts` |
| Pairing code not working | Code expires in 60s. Ensure phone number includes country code without `+`. |
| "conflict" disconnection | Two instances connected. Kill: `pkill -f "node dist/index.js"` then restart. |
| Bot not responding | Check `store/auth/creds.json` exists. Check registration. Check service running. |

## Removal

1. `rm -rf store/auth/`
2. `sqlite3 store/messages.db "DELETE FROM registered_groups WHERE jid LIKE '%@g.us' OR jid LIKE '%@s.whatsapp.net'"`
3. `mkdir -p data/env && cp .env data/env/env`
4. Rebuild and restart
