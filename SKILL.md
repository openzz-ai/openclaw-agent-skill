---
name: openclaw-agent-bootstrap
description: Use when setting up a reusable OpenClaw agent workflow with a dedicated ops agent, aggressive context hygiene, reset handoff rules, and a no-brain START-HERE file. Covers config tuning for main on gpt-5.4, memory-enabled compaction, workspace bootstrap files, and deployment to other OpenClaw machines.
---

# OpenClaw Agent Bootstrap Skill

## Purpose

This skill packages a reusable OpenClaw operating model:

- Keep `main` on `gpt-5.4`
- Prevent long threads from collapsing under context pressure
- Route OpenClaw operational work into a dedicated `openclaw-ops` agent
- Give the operator one no-brain entrypoint: `START-HERE.md`

## What This Skill Produces

- A config snippet for `~/.openclaw/openclaw.json`
- A dedicated `openclaw-ops` agent definition
- A `START-HERE.md` file with minimal routing rules
- `AGENTS.md`, SOP, and runbooks for reset and handoff
- A shareable desktop template folder and zip package

## Recommended Layout

```text
~/.openclaw/agents/openclaw-ops/agent
~/.openclaw/agents/openclaw-ops/sessions
~/.openclaw/workspace-openclaw-ops/
  START-HERE.md
  AGENTS.md
  docs/main-session-sop.md
  runbooks/context-management.md
  runbooks/main-session-reset-template.md
```

## Core Config Strategy

Apply these ideas:

- `agents.defaults.model.primary = openai-codex/gpt-5.4`
- `agents.defaults.contextPruning.ttl = 30m`
- `agents.defaults.compaction.keepRecentTokens = 60000`
- `agents.defaults.compaction.reserveTokens = 25000`
- `agents.defaults.compaction.reserveTokensFloor = 25000`
- `agents.defaults.compaction.model = zai/glm-5`
- `agents.defaults.compaction.memoryFlush.enabled = true`
- `plugins.entries.memory-lancedb-pro.config.autoRecall = true`
- `plugins.entries.memory-lancedb-pro.config.sessionMemory.enabled = true`

See `references/openclaw-config-snippet.json`.

## No-Brain Rules

- Normal chat -> `main`
- OpenClaw/plugins/gateway/context/config/security -> `openclaw-ops`
- Heavy coding -> `codex`
- `main` above `80%` -> `/new`
- `main` at `90%` or higher -> `/new` immediately
- After `/new`, paste only a minimal handoff block

## Handoff Block

```text
Working context from previous thread:

- Objective: continue only the active task
- Confirmed state: prior thread was reset to avoid context overflow
- Files: include only the file paths that still matter
- Next step: continue from the current task only
- Constraint: do not reconstruct the old thread
```

## Workflow

1. Create the `openclaw-ops` agent and workspace
2. Copy the workspace bootstrap files from `references/workspace/`
3. Merge `references/openclaw-config-snippet.json` into `~/.openclaw/openclaw.json`
4. Replace absolute paths for the target machine
5. Validate and restart:

```bash
openclaw config validate
openclaw gateway restart
openclaw gateway status
```

## Success Signals

- `openclaw config validate` passes
- `openclaw gateway status` shows `RPC probe: ok`
- `openclaw status` shows the new `openclaw-ops` agent
- `main` threads are reset instead of being stretched past 90%

## References

- `references/openclaw-config-snippet.json`
- `references/workspace/START-HERE.md`
- `references/workspace/AGENTS.md`
- `references/workspace/docs/main-session-sop.md`
- `references/workspace/runbooks/context-management.md`
- `references/workspace/runbooks/main-session-reset-template.md`
- `references/DEPLOY-CHECKLIST.md`
- `references/使用说明-中文.md`
