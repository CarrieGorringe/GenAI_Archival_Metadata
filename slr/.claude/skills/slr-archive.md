---
name: slr-archive
description: Toggle automated turn-by-turn session archiving on or off for the SLR project. Default is on. Use `/slr-archive on`, `/slr-archive off`, or `/slr-archive status`.
allowed-tools: Read, Edit, Write, Bash(cat *), Bash(bash -n *)
---

# slr-archive

Toggle the automated Stop hook that writes a turn marker to `methodology/chat-history/YYYY-MM-DD_auto.md` after every Claude response.

## Inputs

Argument: `on`, `off`, or `status`. If omitted, treat as `status`.

## Settings file location

The hook lives in `slr/.claude/settings.json` (relative to the project root). Its shape when **enabled**:

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash scripts/auto-archive-hook.sh"
          }
        ]
      }
    ]
  }
}
```

When **disabled**, the `Stop` key is present but the hooks array is empty:

```json
{
  "hooks": {
    "Stop": []
  }
}
```

## Workflow

### `status`
1. Read `slr/.claude/settings.json`.
2. Check whether `hooks.Stop` contains at least one entry with `command` matching `auto-archive-hook.sh`.
3. Report: **on** or **off**, and the path of today's auto log if it exists (`methodology/chat-history/YYYY-MM-DD_auto.md`).

### `on`
1. Read `slr/.claude/settings.json`.
2. If already enabled (hook entry present), report "Already on" and stop.
3. Otherwise set `hooks.Stop` to the full enabled array shown above.
4. Write the file back.
5. Report: "Archiving turned **on**. Turn markers will be written after each response."

### `off`
1. Read `slr/.claude/settings.json`.
2. If already disabled (empty Stop array), report "Already off" and stop.
3. Otherwise set `hooks.Stop` to `[]`.
4. Write the file back.
5. Report: "Archiving turned **off**. Existing logs in `methodology/chat-history/` are untouched."

## Notes

- Never touch `settings.local.json` — that file holds per-machine permissions.
- The hook script itself is at `slr/scripts/auto-archive-hook.sh`; this skill only controls whether it is wired to the Stop event.
- After toggling, tell the user to **restart Claude Code** for the change to take effect.
