# openclaw-agent-skill

A reusable OpenClaw skill/bootstrap kit for keeping `main` on `gpt-5.4`, reducing context overload, routing operations work into `openclaw-ops`, and standardizing reset handoffs.

## Included

- `SKILL.md` - skill definition
- `references/openclaw-config-snippet.json` - config snippet for `~/.openclaw/openclaw.json`
- `references/workspace/` - workspace files for `openclaw-ops`
- `references/DEPLOY-CHECKLIST.md` - deployment checklist
- `references/使用说明-中文.md` - Chinese usage guide

## Quick Start

1. Create an `openclaw-ops` agent and workspace on the target machine.
2. Copy `references/workspace/` into that workspace.
3. Merge `references/openclaw-config-snippet.json` into the target `~/.openclaw/openclaw.json`.
4. Update absolute paths.
5. Run `openclaw config validate`, `openclaw gateway restart`, and `openclaw gateway status`.
