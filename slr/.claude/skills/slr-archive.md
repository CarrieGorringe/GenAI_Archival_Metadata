---
name: slr-archive
description: Toggle automated turn-by-turn session archiving on or off for the SLR project, and generate end-of-session summaries. Default is on. Use `/slr-archive on`, `/slr-archive off`, `/slr-archive summary`, or `/slr-archive status`.
allowed-tools: Read, Edit, Write, Bash(cat *), Bash(bash -n *), Bash(bash slr/scripts/archive-session.sh *), Bash(bash scripts/archive-session.sh *)
---

# slr-archive

Toggle the automated Stop hook that writes a turn marker to `methodology/chat-history/YYYY-MM-DD_auto.md` after every Claude response. Also generates end-of-session summary documents via `archive-session.sh --mode summary`.

## Inputs

Argument: `on`, `off`, `summary`, or `status`. If omitted, treat as `status`.

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
1. Generate a session summary first — follow the full `summary` workflow below.
2. Read `slr/.claude/settings.json`.
3. If already disabled (empty Stop array), report "Already off" (but the summary was still generated).
4. Otherwise set `hooks.Stop` to `[]`.
5. Write the file back.
6. Report: "Archiving turned **off**. Summary written to `<path>`. Restart Claude Code for the hook change to take effect."

### `summary`
1. Ask the user for a short session title (one line). If they provide one as an argument after `summary`, use that directly without asking.
2. Run: `bash scripts/archive-session.sh --mode summary --title "<title>"`
   - If running from outside the `slr/` directory, use the full path: `bash slr/scripts/archive-session.sh --mode summary --title "<title>"`
3. The script prints the path of the created file. Read that file.
4. Fill in the three template sections based on this conversation:
   - **Summary**: one paragraph covering what was accomplished this session
   - **Key decisions**: bullet list of decisions made, each with a brief rationale
   - **Next steps**: what to pick up next session
5. Write the filled-in content back to the file.
6. Report the file path and a one-line summary of what was captured.

## Notes

- Never touch `settings.local.json` — that file holds per-machine permissions.
- The hook script itself is at `slr/scripts/auto-archive-hook.sh`; this skill only controls whether it is wired to the Stop event.
- After toggling on/off, tell the user to **restart Claude Code** for the hook change to take effect.
- `summary` does **not** affect the hook state — archiving stays on after generating a summary.
