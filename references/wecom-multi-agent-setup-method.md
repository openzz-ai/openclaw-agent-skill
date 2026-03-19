# OpenClaw WeCom Multi-Agent Setup Method

This document explains the setup method only. It does not export any machine-specific secrets.

## Goal

- Use WeCom as the primary active channel
- Bind one WeCom bot to one agent
- Keep `main` as the general entry point
- Keep Feishu paused by default and only re-enable it when needed

## What You Need

- One OpenClaw instance
- Multiple agents already created
- One WeCom bot per agent, each with:
  - `botId`
  - `secret`

It is best to keep each `accountId` aligned with the target agent name.

Examples:

- `life`
- `openclaw-ops`
- `brand-cto`
- `brand-archivist`
- `global-sentinel`
- `codex`

## Core Idea

You only need to update two areas in `~/.openclaw/openclaw.json`:

1. `channels.wecom.accounts`
2. `bindings`

## 1) Add WeCom Accounts

Each WeCom bot goes under `channels.wecom.accounts`.

```json
"channels": {
  "wecom": {
    "enabled": true,
    "botId": "primary-entry-botId",
    "secret": "primary-entry-secret",
    "accounts": {
      "<accountId>": {
        "enabled": true,
        "botId": "botId-for-this-agent",
        "secret": "secret-for-this-agent"
      }
    },
    "defaultAccount": "<an-accountId-that-really-exists>"
  }
}
```

Important:

- `defaultAccount` must match a real key inside `accounts`
- do not leave it pointing to a non-existent value

## 2) Add Per-Agent Routes

Each agent gets one route:

```json
{
  "type": "route",
  "agentId": "<agent-id>",
  "match": {
    "channel": "wecom",
    "accountId": "<accountId>"
  }
}
```

## 3) Keep a General `main` Route

It is a good idea to keep a general WeCom route for `main`:

```json
{
  "type": "route",
  "agentId": "main",
  "match": {
    "channel": "wecom"
  }
}
```

This catches unmatched WeCom traffic and sends it to `main`.

## Recommended Mapping Pattern

- `main` -> general WeCom entry
- `life` -> `life-steward`
- `openclaw-ops` -> `openclaw-ops`
- `brand-cto` -> `brand-cto`
- `brand-archivist` -> `brand-archivist`
- `global-sentinel` -> `global-sentinel`
- `codex` -> `codex`

## Minimal Template

```json
{
  "channels": {
    "wecom": {
      "enabled": true,
      "botId": "primary-entry-botId",
      "secret": "primary-entry-secret",
      "accounts": {
        "life": {
          "enabled": true,
          "botId": "life botId",
          "secret": "life secret"
        },
        "openclaw-ops": {
          "enabled": true,
          "botId": "openclaw-ops botId",
          "secret": "openclaw-ops secret"
        },
        "brand-cto": {
          "enabled": true,
          "botId": "brand-cto botId",
          "secret": "brand-cto secret"
        },
        "brand-archivist": {
          "enabled": true,
          "botId": "brand-archivist botId",
          "secret": "brand-archivist secret"
        },
        "global-sentinel": {
          "enabled": true,
          "botId": "global-sentinel botId",
          "secret": "global-sentinel secret"
        },
        "codex": {
          "enabled": true,
          "botId": "codex botId",
          "secret": "codex secret"
        }
      },
      "defaultAccount": "life"
    }
  },
  "bindings": [
    {
      "type": "route",
      "agentId": "main",
      "match": {
        "channel": "wecom"
      }
    },
    {
      "type": "route",
      "agentId": "life-steward",
      "match": {
        "channel": "wecom",
        "accountId": "life"
      }
    },
    {
      "type": "route",
      "agentId": "openclaw-ops",
      "match": {
        "channel": "wecom",
        "accountId": "openclaw-ops"
      }
    },
    {
      "type": "route",
      "agentId": "brand-cto",
      "match": {
        "channel": "wecom",
        "accountId": "brand-cto"
      }
    },
    {
      "type": "route",
      "agentId": "brand-archivist",
      "match": {
        "channel": "wecom",
        "accountId": "brand-archivist"
      }
    },
    {
      "type": "route",
      "agentId": "global-sentinel",
      "match": {
        "channel": "wecom",
        "accountId": "global-sentinel"
      }
    },
    {
      "type": "route",
      "agentId": "codex",
      "match": {
        "channel": "wecom",
        "accountId": "codex"
      }
    }
  ]
}
```

## How To Pause Feishu

If you also want to make WeCom the primary channel and pause Feishu:

```json
{
  "channels": {
    "feishu": {
      "enabled": false
    }
  },
  "plugins": {
    "entries": {
      "openclaw-lark": {
        "enabled": false
      }
    }
  }
}
```

## How To Add One More Agent Later

Only do these two things:

1. add a new `channels.wecom.accounts.<accountId>` entry
2. add a matching `bindings[]` route for that `accountId`

## Required Validation Steps

Run these after every channel change:

```bash
openclaw config validate
openclaw gateway restart
openclaw gateway status
```

Healthy result:

- config validation passes
- gateway restarts successfully
- `RPC probe: ok`

## Common Mistakes

### `defaultAccount` does not exist

Symptom:

- doctor warning says `defaultAccount` does not match configured accounts

Fix:

- set `defaultAccount` to a real key from `accounts`

### Bot is configured but messages do not route

Usually this means:

- you added `channels.wecom.accounts`
- but forgot to add the matching `bindings` route

### Route exists but agent does not respond

Check these first:

- `openclaw gateway status`
- `botId` and `secret`
- `bindings[].agentId` matches the real target agent id

## Recommendations

- one bot per agent is the clearest setup
- keep `main` as the general entry point
- give ops-style agents their own bot when possible
- if Feishu is unstable, pause it instead of letting it share primary-entry duty with WeCom
