# Deploy Checklist

- Create `openclaw-ops` agent directories on the target machine
- Copy `references/workspace/` into the target workspace
- Merge `references/openclaw-config-snippet.json` into target `~/.openclaw/openclaw.json`
- Add an `openclaw-ops` entry to `agents.list`
- Replace absolute paths
- Run:

```bash
openclaw config validate
openclaw gateway restart
openclaw gateway status
```
