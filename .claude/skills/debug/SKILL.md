---
name: debug
description: Debug container agent issues. Use when things aren't working, container fails, authentication problems, or to understand how the container system works. Covers logs, environment variables, mounts, and common issues.
guide: ../../guides/debug/guide.md
---

# NanoClaw Container Debugging

Read the guide at `guides/debug/guide.md` for architecture details, mount structure, common issues, and diagnostic commands. This skill orchestrates debugging via Claude Code tools.

## Debugging Workflow

1. **Identify the symptom** — ask the user what's happening or check recent logs.
2. **Check logs** — read `logs/nanoclaw.log`, `logs/nanoclaw.error.log`, and `groups/{folder}/logs/container-*.log`.
3. **Run diagnostics** — use the quick diagnostic script from the guide.
4. **Match to common issue** — refer to the guide's Common Issues section.
5. **Fix and verify** — apply the fix, rebuild if needed, test.

## Quick Actions

**View recent logs:**
```bash
tail -50 logs/nanoclaw.log
```

**Check container runs:**
```bash
ls -t groups/*/logs/container-*.log | head -5
```

**Read latest container log:**
```bash
cat $(ls -t groups/*/logs/container-*.log | head -1)
```

**Test container manually:**
```bash
mkdir -p data/env groups/test && cp .env data/env/env
echo '{"prompt":"What is 2+2?","groupFolder":"test","chatJid":"test@g.us","isMain":false}' | \
  docker run -i -v $(pwd)/data/env:/workspace/env-dir:ro -v $(pwd)/groups/test:/workspace/group nanoclaw-agent:latest
```

**Check image contents:**
```bash
docker run --rm --entrypoint /bin/bash nanoclaw-agent:latest -c 'node --version && claude --version'
```

Refer to the guide for detailed explanations of each issue and fix.
