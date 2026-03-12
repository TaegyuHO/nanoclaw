---
name: add-pdf-reader
description: Add PDF reading capability using pdftotext. Handles WhatsApp attachments, URLs, and local files.
tags: [feature, pdf, whatsapp]
requires:
  tools: [git, npm, docker]
  prerequisites: [whatsapp-channel]
---

# Add PDF Reader

## Overview

Adds PDF text extraction to container agents via poppler-utils (pdftotext/pdfinfo). PDFs sent as WhatsApp attachments are auto-downloaded to the group workspace.

## Prerequisites

- WhatsApp channel installed (required — this modifies WhatsApp channel files)

## Code Installation

```bash
git remote add whatsapp https://github.com/qwibitai/nanoclaw-whatsapp.git  # if not already added
git fetch whatsapp skill/pdf-reader
git merge whatsapp/skill/pdf-reader || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `container/skills/pdf-reader/` — Agent-facing CLI and docs
- `poppler-utils` in container Dockerfile
- PDF attachment download in WhatsApp channel

Validate, rebuild container, and restart:

```bash
npm run build
npx vitest run src/channels/whatsapp.test.ts
./container/build.sh
```

## Verification

1. Send a PDF in a registered WhatsApp chat — agent should acknowledge it
2. Ask agent to read a PDF URL — it uses `pdf-reader fetch <url>`

Check logs: `tail -f logs/nanoclaw.log | grep -i pdf`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| pdf-reader command not found | Rebuild container: `./container/build.sh` |
| Empty text extraction | PDF may be scanned (image-based). pdftotext only handles text PDFs. |
| WhatsApp PDF not detected | Verify message has `documentMessage` with `mimetype: application/pdf`. |
