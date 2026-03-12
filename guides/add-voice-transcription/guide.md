---
name: add-voice-transcription
description: Add voice message transcription using OpenAI Whisper API. Transcribes WhatsApp voice notes automatically.
tags: [feature, voice, whisper, whatsapp]
requires:
  tools: [git, npm]
  services: [openai-api]
  prerequisites: [whatsapp-channel]
---

# Add Voice Transcription

## Overview

Adds automatic voice message transcription to WhatsApp. When a voice note arrives, it is downloaded, sent to OpenAI Whisper API, and delivered to the agent as `[Voice: <transcript>]`.

**Cost:** ~$0.006 per minute of audio (~$0.003 per typical 30-second voice note).

## Prerequisites

- WhatsApp channel installed (required — this modifies WhatsApp channel files)
- OpenAI API key from [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

## Code Installation

```bash
git remote add whatsapp https://github.com/qwibitai/nanoclaw-whatsapp.git  # if not already added
git fetch whatsapp skill/voice-transcription
git merge whatsapp/skill/voice-transcription || {
  git checkout --theirs package-lock.json
  git add package-lock.json
  git merge --continue
}
```

This adds:
- `src/transcription.ts` — Voice transcription module
- Voice handling in `src/channels/whatsapp.ts`
- `openai` npm dependency

Validate:

```bash
npm install --legacy-peer-deps && npm run build
npx vitest run src/channels/whatsapp.test.ts
```

## Configuration

Add to `.env`:

```bash
OPENAI_API_KEY=sk-your-key
```

Sync: `mkdir -p data/env && cp .env data/env/env`

Build and restart service.

## Verification

Send a voice note in a registered WhatsApp chat. The agent should receive and respond to the transcribed content.

Check logs: `tail -f logs/nanoclaw.log | grep -i voice`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "[Voice Message - transcription unavailable]" | Check `OPENAI_API_KEY` in `.env` and `data/env/env`. Verify key: `curl -s https://api.openai.com/v1/models -H "Authorization: Bearer $OPENAI_API_KEY"` |
| "[Voice Message - transcription failed]" | Check OpenAI billing. Regenerate key if invalid. |
| Agent doesn't respond to voice notes | Verify chat is registered and service is running. |
