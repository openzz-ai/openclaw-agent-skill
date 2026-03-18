# openclaw-agent-skill

A reusable OpenClaw skill/bootstrap kit for keeping `main` on `gpt-5.4`, reducing context overload, routing operations work into `openclaw-ops`, and standardizing reset handoffs.

## Contents / 目录

- [简体中文](#简体中文)
- [中文快速开始](#中文快速开始)
- [What Problem This Solves](#what-problem-this-solves)
- [Included](#included)
- [Repo Layout](#repo-layout)
- [Quick Start](#quick-start)
- [Core Behavior](#core-behavior)
- [No-Brain Routing](#no-brain-routing)
- [Example: When To Leave `main`](#example-when-to-leave-main)
- [Example: `/new` Handoff](#example-new-handoff)
- [Install Checklist](#install-checklist)
- [Notes](#notes)

## 简体中文

这是一个可复用的 OpenClaw skill / bootstrap kit，用来解决这几类常见问题：

- `main` 会话越聊越长，最后把上下文顶满
- OpenClaw 运维问题和普通聊天混在一起
- `/new` 用得太晚，或者重开后没有标准 handoff
- 不同机器上的 agent 规则不一致

这套仓库的目标是提供一套可复制的工作方式：

- 保持 `main` 使用 `gpt-5.4`
- 用 compaction + memory 控制上下文膨胀
- 把 OpenClaw 配置、插件、网关、上下文治理分流到 `openclaw-ops`
- 用一个 `START-HERE.md` 实现“无脑操作”

快速理解：

- 普通聊天 -> `main`
- OpenClaw/插件/网关/上下文/配置 -> `openclaw-ops`
- 大代码任务 -> `codex`
- `main` 超过 `80%` -> `/new`
- `main` 到 `90%` 或更高 -> 立即 `/new`

## 中文快速开始

1. 在目标机器上创建一个 `openclaw-ops` agent 和对应 workspace。
2. 把 `references/workspace/` 复制到这个 workspace。
3. 把 `references/openclaw-config-snippet.json` 合并进目标机器的 `~/.openclaw/openclaw.json`。
4. 修改其中的绝对路径配置。
5. 执行下面三条命令验证配置是否生效：

```bash
openclaw config validate
openclaw gateway restart
openclaw gateway status
```

如果结果正常，你应该能看到：

- 配置校验通过
- 网关重启成功
- `RPC probe: ok`
- `openclaw status` 里出现 `openclaw-ops`

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
