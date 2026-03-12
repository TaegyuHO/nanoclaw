---
name: add-image-vision
description: Add image understanding to agents. Resizes and processes WhatsApp image attachments as multimodal content.
tags: [feature, image, vision, whatsapp]
requires:
  tools: [git, npm, docker]
  prerequisites: [whatsapp-channel]
---

# Add Image Vision

## Overview

Enables agents to see and understand images sent via WhatsApp. Images are downloaded, resized with sharp, and passed to Claude as base64-encoded multimodal content blocks.

## Prerequisites

- WhatsApp channel installed (required — this modifies WhatsApp channel files)
- Build tools for sharp native bindings (`xcode-select --install` on macOS, `build-essential` on Linux)

## Code Installation

```bash
git remote add whatsapp https://github.com/qwibitai/nanoclaw-whatsapp.git  # if not already added
git fetch whatsapp skill/image-vision
git merge whatsapp/skill/image-vision || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/image.ts` — Image download, resize, base64 encoding
- Image handling in WhatsApp channel and container agent-runner
- `sharp` dependency

Validate:

```bash
npm install && npm run build
npx vitest run src/image.test.ts
```

Rebuild container and sync agent-runner:

```bash
./container/build.sh
for dir in data/sessions/*/agent-runner-src/; do
  cp container/agent-runner/src/*.ts "$dir"
done
```

Restart service.

## Verification

Send an image in a registered WhatsApp group. The agent should respond with understanding of the image content.

Check logs: `tail -50 groups/*/logs/container-*.log`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Image - download failed" | Check WhatsApp connection stability. |
| "Image - processing failed" | Verify sharp: `npm ls sharp`. Reinstall if needed. |
| Agent doesn't mention image | Ensure agent-runner source was synced to group caches. |
