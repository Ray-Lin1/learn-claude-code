# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 用户设置

**请称呼我为 "ray"** - 在回复中请使用这个名字称呼用户。

## Repository Purpose

This is a **teaching repository** for agent harness engineering. It reverse-engineers Claude Code's architecture into 12 progressive sessions (s01-s12), demonstrating how to build the environment around an AI agent model (the "harness") rather than the agent itself.

**Core philosophy**: The model (Claude) is the agent. The code is the harness. This repo teaches you to build vehicles for intelligence, not intelligence itself.

## Running Sessions

Each session in `agents/` is a self-contained, runnable Python script that demonstrates one harness mechanism:

```bash
# Setup
pip install -r requirements.txt
cp .env.example .env  # Edit .env with your ANTHROPIC_API_KEY

# Run any session (progressive: s01 -> s12)
python agents/s01_agent_loop.py       # The agent loop (start here)
python agents/s02_tool_use.py         # Tool dispatch
python agents/s03_todo_write.py       # Task planning
python agents/s04_subagent.py         # Subagent spawning
python agents/s05_skill_loading.py    # On-demand knowledge
python agents/s06_context_compact.py  # Context compression
python agents/s07_task_system.py      # Persistent tasks
python agents/s08_background_tasks.py # Background execution
python agents/s09_agent_teams.py      # Multi-agent teams
python agents/s10_team_protocols.py   # Team coordination
python agents/s11_autonomous_agents.py # Autonomous agents
python agents/s12_worktree_task_isolation.py  # Worktree isolation
python agents/s_full.py               # All mechanisms combined (capstone)
```

## Web Platform

Interactive documentation with step-through diagrams and code viewer:

```bash
cd web
npm install
npm run dev      # Development server at http://localhost:3000
npm run build    # Production build
npx tsc --noEmit # Type checking
```

## Session Progression

The 12 sessions follow a deliberate progression from a minimal loop to full multi-agent systems:

**Phase 1: THE LOOP** (s01-s02)
- s01: The agent loop (`while stop_reason == "tool_use"`)
- s02: Tool dispatch (name → handler mapping)

**Phase 2: PLANNING & KNOWLEDGE** (s03-s06)
- s03: TodoWrite (plan before executing)
- s04: Subagents (fresh context per subtask)
- s05: Skills (load knowledge on-demand via tool_result)
- s06: Context compression (3-layer compression strategy)

**Phase 3: PERSISTENCE** (s07-s08)
- s07: Task system (file-based CRUD with dependency graph)
- s08: Background tasks (daemon threads + notification queue)

**Phase 4: TEAMS** (s09-s12)
- s09: Agent teams (teammates + JSONL mailboxes)
- s10: Team protocols (request-response negotiation)
- s11: Autonomous agents (idle cycle + auto-claim)
- s12: Worktree isolation (task coordination + isolated execution)

## Core Agent Loop Pattern

Every session implements this fundamental pattern:

```python
def agent_loop(messages):
    while True:
        response = client.messages.create(
            model=MODEL, system=SYSTEM, messages=messages, tools=TOOLS
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason != "tool_use":
            return  # Model finished

        # Execute tools and append results
        results = []
        for block in response.content:
            if block.type == "tool_use":
                output = TOOL_HANDLERS[block.name](**block.input)
                results.append({"type": "tool_result", "tool_use_id": block.id, "content": output})
        messages.append({"role": "user", "content": results})
```

The loop is invariant. Sessions layer mechanisms on top without changing it.

## Architecture Notes

### Self-Contained Sessions
Each `agents/s*.py` file is **fully self-contained**:
- No shared imports between sessions
- All mechanisms reimplemented per session for clarity
- Can run any session independently without others

### State Persistence
Sessions persist runtime state in gitignored directories:
- `.tasks/` - Task JSON files with dependency graph
- `.teams/` - Teammate mailboxes (JSONL)
- `.task_outputs/` - Background task outputs
- `.transcripts/` - Compressed conversation history

### Mental-Model-First Documentation
The `docs/{en,zh,ja}/` directory contains session documentation organized by:
- Problem → Solution → ASCII Diagram → Minimal Code
- Read these before modifying sessions to understand intent

## Dependencies

**Python**:
- `anthropic>=0.25.0` - Anthropic API client
- `python-dotenv>=1.0.0` - Environment configuration

**Web Platform**:
- Next.js 16 + React 19
- TypeScript
- Tailwind CSS 4
- Framer Motion (animations)

## Development Workflow

When working with this codebase:

1. **Understand the session scope** - Each session adds ONE mechanism. Don't add beyond its scope.
2. **Preserve self-containment** - Keep sessions independent. No cross-session imports.
3. **Test interactively** - Sessions are designed for interactive REPL testing.
4. **Update docs** - If changing session behavior, update corresponding `docs/{en,zh,ja}/s*.md` file.
5. **Web platform** - Content extracted from docs via `npm run extract`. Rebuild after doc changes.

## CI/CD

- `.github/workflows/ci.yml` - Typecheck + build web/ on push to main
- `.github/workflows/test.yml` - Unit and session tests (requires secrets)
