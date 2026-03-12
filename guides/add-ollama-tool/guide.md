---
name: add-ollama-tool
description: Add Ollama MCP server so agents can call local models for cheaper/faster tasks.
tags: [feature, ollama, local-models, mcp]
requires:
  tools: [git, npm, docker, ollama]
---

# Add Ollama Integration

## Overview

Adds a stdio-based MCP server exposing local Ollama models as tools. Claude remains the orchestrator but can offload work (summarization, translation, general queries) to local models.

Tools added: `ollama_list_models`, `ollama_generate`.

## Prerequisites

- Ollama installed and running: [ollama.com/download](https://ollama.com/download)
- At least one model pulled:

```bash
ollama pull gemma3:1b       # Small, fast (1GB)
ollama pull llama3.2         # Good general purpose (2GB)
ollama pull qwen3-coder:30b  # Best for code tasks (18GB)
```

## Code Installation

```bash
git fetch upstream skill/ollama-tool
git merge upstream/skill/ollama-tool
```

This adds:
- `container/agent-runner/src/ollama-mcp-stdio.ts` — Ollama MCP server
- `scripts/ollama-watch.sh` — macOS notification watcher
- Ollama MCP config in agent-runner

Copy to existing group caches:

```bash
for dir in data/sessions/*/agent-runner-src; do
  cp container/agent-runner/src/ollama-mcp-stdio.ts "$dir/"
  cp container/agent-runner/src/index.ts "$dir/"
done
```

Build: `npm run build && ./container/build.sh`

## Configuration

Default host: `http://host.docker.internal:11434` (Docker Desktop). Custom:

```bash
# In .env
OLLAMA_HOST=http://your-host:11434
```

Restart service.

## Verification

Send: "use ollama to tell me the capital of France"

Monitor: `tail -f logs/nanoclaw.log | grep -i ollama`

Optional macOS notifications: `./scripts/ollama-watch.sh`

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Ollama is not installed" | Agent using CLI instead of MCP tools. Verify MCP config in agent-runner, re-copy files, rebuild container. |
| "Failed to connect" | Verify Ollama running: `ollama list`. Test Docker→host: `docker run --rm curlimages/curl curl -s http://host.docker.internal:11434/api/tags` |
| Agent doesn't use Ollama | Be explicit: "use the ollama_generate tool with gemma3:1b..." |
