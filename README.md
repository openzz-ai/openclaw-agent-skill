# openclaw-agent-skill

A reusable OpenClaw skill/bootstrap kit for keeping `main` on `gpt-5.4`, reducing context overload, routing operations work into `openclaw-ops`, and standardizing reset handoffs.

## What Problem This Solves

OpenClaw setups often drift into one overloaded `main` thread:

- long chats keep growing until context quality drops
- operations work mixes with normal chat
- `/new` happens too late or with no handoff discipline
- each machine ends up with different rules

This repo gives you a repeatable operating model:

- keep `main` on `gpt-5.4`
- compact aggressively enough to stay healthy
- move OpenClaw ops work into a dedicated `openclaw-ops` agent
- use a single `START-HERE.md` file for no-brain routing

## Included

- `SKILL.md` - skill definition
- `references/openclaw-config-snippet.json` - config snippet for `~/.openclaw/openclaw.json`
- `references/workspace/` - workspace files for `openclaw-ops`
- `references/DEPLOY-CHECKLIST.md` - deployment checklist
- `references/使用说明-中文.md` - Chinese usage guide

## Repo Layout

```text
.
├── SKILL.md
├── references/
│   ├── openclaw-config-snippet.json
│   ├── DEPLOY-CHECKLIST.md
│   ├── 使用说明-中文.md
│   └── workspace/
│       ├── START-HERE.md
│       ├── AGENTS.md
│       ├── docs/main-session-sop.md
│       └── runbooks/
│           ├── context-management.md
│           └── main-session-reset-template.md
```

## Quick Start

1. Create an `openclaw-ops` agent and workspace on the target machine.
2. Copy `references/workspace/` into that workspace.
3. Merge `references/openclaw-config-snippet.json` into the target `~/.openclaw/openclaw.json`.
4. Update absolute paths.
5. Run `openclaw config validate`, `openclaw gateway restart`, and `openclaw gateway status`.

## Core Behavior

This kit assumes:

- `main` stays on `openai-codex/gpt-5.4`
- compaction keeps `60000` recent tokens
- compaction reserves `25000` tokens of headroom
- compaction summaries use `zai/glm-5`
- old tool-result context is pruned with `30m` TTL
- memory recall and session memory are enabled through `memory-lancedb-pro`

## No-Brain Routing

The shortest rulebook lives in `references/workspace/START-HERE.md`:

- normal chat -> `main`
- OpenClaw/plugins/gateway/context/config/security -> `openclaw-ops`
- heavy coding -> `codex`
- `main` above `80%` -> `/new`
- `main` at `90%` or higher -> `/new` immediately

## Example: When To Leave `main`

Stay in `main` when:

- the topic is short-lived
- the conversation is mixed or exploratory
- you do not expect many follow-up turns

Move to `openclaw-ops` when:

- the topic is OpenClaw config
- the issue involves plugins, gateway, auth, channels, or context limits
- the thread is becoming a reusable operational procedure

Move to `codex` when:

- the work is implementation-heavy
- the task will involve multiple file edits, tests, and iteration

## Example: `/new` Handoff

Use this after resetting an overloaded `main` thread:

```text
Working context from previous thread:

- Objective: continue only the active task
- Confirmed state: prior thread was reset to avoid context overflow
- Files: include only the file paths that still matter
- Next step: continue from the current task only
- Constraint: do not reconstruct the old thread
```

## Install Checklist

After merging the config and copying workspace files:

```bash
openclaw config validate
openclaw gateway restart
openclaw gateway status
```

Healthy result:

- config validates cleanly
- gateway restarts successfully
- `RPC probe: ok`
- `openclaw-ops` appears in `openclaw status`

## Notes

- This repo is a bootstrap kit, not a secrets bundle
- absolute paths must be customized on each destination machine
- `references/使用说明-中文.md` is the fastest Chinese overview
