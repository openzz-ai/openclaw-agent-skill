# Context Management Runbook

Quick entrypoint: `START-HERE.md`

## Goal

Keep `main` on `gpt-5.4` without letting long-running conversations hit the 128k context wall.

## Current Policy

- `main` stays on `openai-codex/gpt-5.4`
- Auto-compaction keeps the most recent `60000` tokens
- Compaction headroom is `25000` tokens
- Tool-result cache TTL is `30m`
- Memory flush runs before compaction
- Compaction summaries use `zai/glm-5`
- Session memory and auto-recall are enabled through `memory-lancedb-pro`

## Trigger Thresholds

- Below `50%`: no action needed
- Around `60-75%`: finish the current subtask, then consider `/new` or moving the thread
- Above `80%`: stop extending the thread casually; summarize and continue in a fresh session
- Above `90%`: reset immediately after capturing durable notes
