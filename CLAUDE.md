# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Autonomous Research Agents built with the Claude Agent SDK (`claude-agent-sdk`). Five progressive demos: single agent → multi-agent orchestration → n8n HTTP API → plan-and-execute with reflection → multi-agent plan-and-reflect. All agents research topics via web search and produce structured Markdown reports.

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
curl -X POST http://localhost:8000/research -H "Content-Type: application/json" -d '{"topic":"Topic","mode":"plan-reflect"}'
curl http://localhost:8000/health

# Demo 4: Plan-and-Execute + Reflection (structured phases via query())
python3 plan_reflect_agent.py "Topic Name"

# Demo 5: Multi-Agent + Plan-and-Execute + Reflection (ClaudeSDKClient)
python3 plan_reflect_multi_agent.py "Topic Name"

# Comparison runner: Demo 1 vs Demo 4 side-by-side
python3 run_comparison.py "Topic Name"
```

No test suite, linter, or build step exists. Testing is manual via CLI runs.

## Architecture

**Five SDK patterns demonstrated:**

- **Demo 1** (`research_agent.py`): `query()` one-shot — single agent with `WebSearch`/`WebFetch` tools, 30 max turns
- **Demo 2** (`multi_agent_research.py`): `ClaudeSDKClient` streaming — orchestrator decomposes topic into subtopics, spawns parallel researcher sub-agents via `AgentDefinition`, stitches results. Orchestrator: 60 max turns, sub-agents: 20 max turns
- **Demo 3** (`n8n_hybrid_server.py`): Starlette/Uvicorn HTTP server wrapping `query()`. Endpoints: `POST /research` (supports `mode=basic|plan-reflect`), `GET /health`. Companion `n8n_workflow.json` for webhook→research→email workflow
- **Demo 4** (`plan_reflect_agent.py`): `query()` with structured 3-phase system prompt — Plan (3-5 research steps) → Execute (sequential with per-step evaluation) → Reflect (self-critique, max 1 correction). Output includes plan, report, reflection notes, and meta-info. 40 max turns
- **Demo 5** (`plan_reflect_multi_agent.py`): `ClaudeSDKClient` combining multi-agent orchestration with plan-and-execute + reflection. Orchestrator plans, delegates steps to parallel sub-agents, synthesizes, and reflects. 60 max turns orchestrator, 20 max turns sub-agents

**Shared code (`utils.py`):**
- `slugify()` — filesystem-safe filename generation
- `strip_preamble()` — removes non-Markdown artifacts from agent output
- `BASIC_SYSTEM_PROMPT` — system prompt for Demos 1, 3 (basic mode)
- `PLAN_REFLECT_SYSTEM_PROMPT` — system prompt for Demos 3 (plan-reflect mode), 4

**Shared patterns across all scripts:**
- Model: `claude-sonnet-4-6`
- Tools: `WebSearch`, `WebFetch` (Claude Code built-in names, not snake_case)
- `permission_mode="bypassPermissions"` for unattended operation
- Auth handled by Claude CLI — no `ANTHROPIC_API_KEY` needed
- All entry points are `async def` with `asyncio.run()`
- Cost extracted from `ResultMessage.total_cost_usd`
- Reports saved to `output/` with pattern-specific prefixes

## Dependencies

Python 3.12+ — install via `pip install -r requirements.txt`.

## n8n Integration

- `n8n_workflow.json`: importable workflow (webhook trigger → HTTP request → format → email)
- `n8nac-config.json`: n8n automation CLI config pointing to local instance at `172.31.224.1:5678`
