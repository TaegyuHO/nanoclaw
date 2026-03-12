---
name: debug
description: Debug containerized agent execution — architecture, logs, mounts, environment variables, sessions, IPC, and common issues.
tags: [core, debug, troubleshooting, container]
requires:
  tools: [docker]
---

# NanoClaw Container Debugging Guide

## Architecture Overview

```
Host                                  Container (Linux VM)
─────────────────────────────────────────────────────────────
src/container-runner.ts               container/agent-runner/
    │                                      │
    │ spawns container                     │ runs Claude Agent SDK
    │ with volume mounts                   │ with MCP servers
    │                                      │
    ├── data/env/env ──────────────> /workspace/env-dir/env
    ├── groups/{folder} ───────────> /workspace/group
    ├── data/ipc/{folder} ────────> /workspace/ipc
    ├── data/sessions/{folder}/.claude/ ──> /home/node/.claude/
    └── (main only) project root ──> /workspace/project
```

The container runs as user `node` (uid 1000) with `HOME=/home/node`. Session files must be mounted to `/home/node/.claude/` (not `/root/.claude/`).

## Log Locations

| Log | Location | Content |
|-----|----------|---------|
| Main app logs | `logs/nanoclaw.log` | Routing, container spawning |
| Main app errors | `logs/nanoclaw.error.log` | Host-side errors |
| Container run logs | `groups/{folder}/logs/container-*.log` | Per-run: input, mounts, stderr, stdout |

Enable debug logging with `LOG_LEVEL=debug`:

```bash
LOG_LEVEL=debug npm run dev
```

## Container Mount Structure

```
/workspace/
├── env-dir/env           # Auth credentials (readonly)
├── group/                # Current group folder (writable)
├── project/              # Project root (main channel only, readonly)
├── global/               # Global CLAUDE.md (non-main, readonly)
├── ipc/                  # Inter-process communication (writable)
│   ├── messages/         # Outgoing messages
│   ├── tasks/            # Scheduled task commands
│   ├── current_tasks.json
│   └── available_groups.json
└── extra/                # Additional custom mounts
```

## Common Issues

### 1. "Claude Code process exited with code 1"

Check the container log: `groups/{folder}/logs/container-*.log`

**Missing authentication:**
```
Invalid API key · Please run /login
```
Ensure `.env` contains `CLAUDE_CODE_OAUTH_TOKEN=sk-ant-oat01-...` or `ANTHROPIC_API_KEY=sk-ant-api03-...`.

**Root user restriction:**
```
--dangerously-skip-permissions cannot be used with root/sudo privileges
```
Container must run as non-root. Check Dockerfile has `USER node`.

### 2. Environment Variables Not Passing

Environment variables passed via `-e` may be lost with `-i` (piped stdin). The system extracts auth variables from `.env` and mounts them at `/workspace/env-dir/env` for sourcing.

Verify:
```bash
echo '{}' | docker run -i \
  -v $(pwd)/data/env:/workspace/env-dir:ro \
  --entrypoint /bin/bash nanoclaw-agent:latest \
  -c 'export $(cat /workspace/env-dir/env | xargs); echo "OAuth: ${#CLAUDE_CODE_OAUTH_TOKEN} chars, API: ${#ANTHROPIC_API_KEY} chars"'
```

### 3. Mount Issues

Check mounts inside container:
```bash
docker run --rm --entrypoint /bin/bash nanoclaw-agent:latest -c 'ls -la /workspace/'
```

Mount syntax: `-v /path:/container/path:ro` (readonly) or `-v /path:/container/path` (read-write).

### 4. Permission Issues

The container runs as `node` (uid 1000). Verify:
```bash
docker run --rm --entrypoint /bin/bash nanoclaw-agent:latest -c '
  whoami
  ls -la /workspace/
  ls -la /app/
'
```

All of `/workspace/` and `/app/` should be owned by `node`.

### 5. Session Not Resuming

The SDK looks for sessions at `$HOME/.claude/projects/`. Inside the container: `/home/node/.claude/projects/`.

Verify mount path:
```bash
grep -A3 "Claude sessions" src/container-runner.ts
```

Test session accessibility:
```bash
docker run --rm --entrypoint /bin/bash \
  -v ~/.claude:/home/node/.claude \
  nanoclaw-agent:latest -c '
echo "HOME=$HOME"
ls -la $HOME/.claude/projects/ 2>&1 | head -5
'
```

### 6. MCP Server Failures

If an MCP server fails to start, the agent may exit. Check container logs for MCP initialization errors.

## Manual Testing

### Full agent flow:
```bash
mkdir -p data/env groups/test
cp .env data/env/env

echo '{"prompt":"What is 2+2?","groupFolder":"test","chatJid":"test@g.us","isMain":false}' | \
  docker run -i \
  -v $(pwd)/data/env:/workspace/env-dir:ro \
  -v $(pwd)/groups/test:/workspace/group \
  -v $(pwd)/data/ipc:/workspace/ipc \
  nanoclaw-agent:latest
```

### Claude Code directly:
```bash
docker run --rm --entrypoint /bin/bash \
  -v $(pwd)/data/env:/workspace/env-dir:ro \
  nanoclaw-agent:latest -c '
  export $(cat /workspace/env-dir/env | xargs)
  claude -p "Say hello" --dangerously-skip-permissions --allowedTools ""
'
```

### Interactive shell:
```bash
docker run --rm -it --entrypoint /bin/bash nanoclaw-agent:latest
```

## SDK Options Reference

```typescript
query({
  prompt: input.prompt,
  options: {
    cwd: '/workspace/group',
    allowedTools: ['Bash', 'Read', 'Write', ...],
    permissionMode: 'bypassPermissions',
    allowDangerouslySkipPermissions: true,
    settingSources: ['project'],
    mcpServers: { ... }
  }
})
```

`allowDangerouslySkipPermissions: true` is required with `permissionMode: 'bypassPermissions'`.

## Rebuilding

```bash
npm run build                    # Rebuild main app
./container/build.sh             # Rebuild container
docker builder prune -af         # Force clean rebuild
./container/build.sh
```

## Session Management

Sessions are stored per-group in `data/sessions/{group}/.claude/`.

```bash
# Clear all sessions
rm -rf data/sessions/

# Clear one group
rm -rf data/sessions/{groupFolder}/.claude/
sqlite3 store/messages.db "DELETE FROM sessions WHERE group_folder = '{groupFolder}'"

# Check session continuity
grep "Session initialized" logs/nanoclaw.log | tail -5
```

## IPC Debugging

```bash
ls -la data/ipc/messages/        # Pending outgoing messages
ls -la data/ipc/tasks/           # Pending task operations
cat data/ipc/{groupFolder}/current_tasks.json
```

**IPC file types:**
- `messages/*.json` — Agent writes: outgoing messages
- `tasks/*.json` — Agent writes: task operations (schedule, pause, resume, cancel)
- `current_tasks.json` — Host writes: read-only task snapshot
- `available_groups.json` — Host writes: group list (main only)

## Quick Diagnostic

```bash
echo "=== NanoClaw Container Diagnostic ==="

echo -e "\n1. Authentication?"
[ -f .env ] && (grep -q "CLAUDE_CODE_OAUTH_TOKEN=sk-" .env || grep -q "ANTHROPIC_API_KEY=sk-" .env) && echo "OK" || echo "MISSING"

echo -e "\n2. Container runtime?"
docker info &>/dev/null && echo "OK" || echo "NOT RUNNING"

echo -e "\n3. Container image?"
echo '{}' | docker run -i --entrypoint /bin/echo nanoclaw-agent:latest "OK" 2>/dev/null || echo "MISSING - run ./container/build.sh"

echo -e "\n4. Session mount path?"
grep -q "/home/node/.claude" src/container-runner.ts 2>/dev/null && echo "OK" || echo "WRONG"

echo -e "\n5. Recent container logs?"
ls -t groups/*/logs/container-*.log 2>/dev/null | head -3 || echo "No logs yet"
```
