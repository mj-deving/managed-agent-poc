# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Autonomous Research Agents built with the Claude Agent SDK (`claude-agent-sdk`). Three progressive demos: single agent → multi-agent orchestration → n8n HTTP API integration. All agents research topics via web search and produce structured Markdown reports.

## Running the Demos

```bash
# Activate venv first
source venv/bin/activate

# Demo 1: Single agent (one-shot via query())
python3 research_agent.py "Topic Name"
python3 research_agent.py "Topic Name" -o reports/

# Demo 2: Multi-agent orchestrator (streaming via ClaudeSDKClient)
python3 multi_agent_research.py "Topic Name"

# Demo 3: HTTP API server for n8n/webhook integration
python3 n8n_hybrid_server.py --port 8000
curl -X POST http://localhost:8000/research -H "Content-Type: application/json" -d '{"topic":"Topic"}'
curl http://localhost:8000/health

# Demo 4: Plan-and-Execute + Reflection (structured phases via query())
python3 plan_reflect_agent.py "Topic Name"
```

No test suite, linter, or build step exists. Testing is manual via CLI runs.

## Architecture

**Three SDK patterns demonstrated:**

- **Demo 1** (`research_agent.py`): `query()` one-shot — single agent with `WebSearch`/`WebFetch` tools, 30 max turns
- **Demo 2** (`multi_agent_research.py`): `ClaudeSDKClient` streaming — orchestrator decomposes topic into subtopics, spawns parallel researcher sub-agents via `AgentDefinition`, stitches results. Orchestrator: 60 max turns, sub-agents: 20 max turns
- **Demo 3** (`n8n_hybrid_server.py`): Starlette/Uvicorn HTTP server wrapping Demo 1's `query()` pattern. Endpoints: `POST /research`, `GET /health`. Companion `n8n_workflow.json` for webhook→research→email workflow
- **Demo 4** (`plan_reflect_agent.py`): `query()` with structured 3-phase system prompt — Plan (3-5 research steps) → Execute (sequential with per-step evaluation) → Reflect (self-critique, max 1 correction). Output includes plan, report, reflection notes, and meta-info. 40 max turns

**Shared patterns across all scripts:**
- Model: `claude-sonnet-4-6`
- Tools: `WebSearch`, `WebFetch` (Claude Code built-in names, not snake_case)
- `permission_mode="bypassPermissions"` for unattended operation
- Auth handled by Claude CLI — no `ANTHROPIC_API_KEY` needed
- `slugify()` utility duplicated in each script for filename sanitization
- All entry points are `async def` with `asyncio.run()`
- Cost extracted from `ResultMessage.total_cost_usd`
- Reports saved to `output/{slug}.md` (or `output/multi-{slug}.md` for Demo 2)

## Dependencies

Python 3.12+ with: `claude-agent-sdk`, `anthropic`, `starlette`, `uvicorn`. No requirements.txt — install manually via pip.

## n8n Integration

- `n8n_workflow.json`: importable workflow (webhook trigger → HTTP request → format → email)
- `n8nac-config.json`: n8n automation CLI config pointing to local instance at `172.31.224.1:5678`
