# Main Session SOP

Quick entrypoint: `START-HERE.md`

1. Use `main` only for short-lived or mixed-topic conversation.
2. If a topic becomes recurring or operational, move it out of `main`.
3. If `main` goes above `80%` context usage, prepare a handoff and use `/new`.
4. Do not carry old tool output into the new thread unless the file path is required.

Routing:

- OpenClaw / gateway / plugins / context limits -> `openclaw-ops`
- Heavy implementation / coding execution -> `codex`
- Fresh unrelated topic -> new thread in `main`
