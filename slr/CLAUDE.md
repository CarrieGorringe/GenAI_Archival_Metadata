# SLR: GenAI for Archival Metadata Extraction

This folder contains the Systematic Literature Review (SLR) on using generative AI to extract structured metadata from unstructured archival and descriptive prose.

## Project context

See the root [README.md](../README.md) for the full project overview. The SLR runs alongside the practical extraction work and informs both pipeline design and evaluation methodology.

**Research question (working):** To what extent can generative AI models reliably extract structured metadata from free-form archival and descriptive prose, and what factors (model, prompt strategy, schema complexity, domain, text type) affect extraction quality?

## Folder layout

```text
slr/
  CLAUDE.md                        ← this file; scoped guidance for working in this directory
  .env.example                     ← Zotero API key template (commit this)
  .env                             ← your actual keys (gitignored — never commit)
  .claude/
    settings.json                  ← project-level Claude Code settings (Stop hook, on by default)
    settings.local.json            ← per-machine permissions (gitignored)
    skills/
      slr-archive.md               ← /slr-archive skill for toggling auto-archiving
  methodology/
    chat-history/                  ← auto session logs + manual checkpoints + end-of-session summaries
    ...                            ← SLR protocol, search strategy, PRISMA notes, screening criteria
  scripts/                         ← reusable, parameterized scripts for the SLR
```

## Chat history & session archiving

Every working session in this directory is archived to `methodology/chat-history/` so decisions and reasoning are traceable across sessions. Archiving is **automated by default** — a Claude Code hook fires after every response and writes a turn marker to a daily rolling log. A summary skill generates an end-of-session narrative on demand.

---

### Automated archiving (default — on)

A `Stop` hook runs `scripts/auto-archive-hook.sh` after every Claude response. It appends a timestamped turn marker to:

```text
methodology/chat-history/YYYY-MM-DD_auto.md
```

The file is created on the first turn of each day and accumulates markers for the whole day's work. Each marker is a placeholder — fill in notes after the turn or let `/slr-archive summary` populate the summary at the end of the session.

#### Controlling auto-archiving with `/slr-archive`

The `/slr-archive` skill manages the hook. Restart Claude Code after any `on`/`off` toggle for the change to take effect.

| Command | What it does |
| --- | --- |
| `/slr-archive status` | Reports whether archiving is on or off; shows today's log path if it exists |
| `/slr-archive on` | Enables the Stop hook (restores it if previously disabled) |
| `/slr-archive off` | Generates an end-of-session summary, then disables the hook |
| `/slr-archive summary` | Generates an end-of-session summary without changing hook state |
| `/slr-archive summary "My title"` | Same, with a specific title instead of a prompt |

#### End-of-session summary

`/slr-archive summary` (or `off`) runs `archive-session.sh --mode summary` and asks Claude to populate the three template sections from the current conversation:

- **Summary** — one paragraph: what was accomplished
- **Key decisions** — bullet list with rationale for each
- **Next steps** — what to pick up next session

The summary file is written to `methodology/chat-history/YYYY-MM-DD_HHMM_<slug>_summary.md`.

**Best practice:** run `/slr-archive summary` (or `/slr-archive off`) while the session is still in context — before it is compressed or a new session starts.

---

### Manual archiving (optional supplement)

Auto-archiving creates structural markers; manual checkpoints let you narrate what happened in detail. Use the archive script directly when you want a richer mid-session record.

#### Checkpoint (every ~30 minutes or ~20–25 turns)

```bash
bash slr/scripts/archive-session.sh --mode checkpoint --title "Brief description"
```

The script creates `methodology/chat-history/YYYY-MM-DD_HHMM_<slug>.md`. After running it, add:

- The user prompt (abbreviated if long)
- What Claude did or decided
- Any output files created or changed

#### End-of-session summary (manual alternative)

```bash
bash slr/scripts/archive-session.sh --mode summary --title "Session description"
```

Then fill in the three template sections manually.

---

### File naming

| Pattern | Source |
| --- | --- |
| `YYYY-MM-DD_auto.md` | Auto-archive hook (daily rolling log) |
| `YYYY-MM-DD_HHMM_<slug>.md` | Manual checkpoint |
| `YYYY-MM-DD_HHMM_<slug>_summary.md` | End-of-session summary (manual or via `/slr-archive summary`) |

Sort by name for chronological order. Do not rename or reorganise files once written — they are a permanent audit trail.

---

## Model selection

Not every task needs the most powerful model. Match the model to the task to keep sessions fast and focused:

| Task type | Recommended model | Why |
| --- | --- | --- |
| Quick reformatting, short lookups, citation cleanup | **Haiku 4.5** (`claude-haiku-4-5`) | Fast; cheap; sufficient for mechanical tasks |
| Most SLR work: synthesis, writing, analysis, screening, script writing | **Sonnet 4.6** (`claude-sonnet-4-6`) | Strong reasoning; good balance of speed and depth |
| Deep methodological reasoning, adversarial review, full-synthesis drafting, complex evaluation design | **Opus 4.8** (`claude-opus-4-8`) | Highest capability; use when quality matters more than speed |

**At the start of each session, state what you are working on** so the right model can be selected. Example: "Today I'm formatting the screening spreadsheet" → Haiku. "Today I'm writing the synthesis section" → Sonnet or Opus.

If the task shifts mid-session (e.g. you move from tagging to writing), flag it so the model can be switched.

---

## Brainstorming guidance

When brainstorming in this directory — search terms, inclusion/exclusion criteria, synthesis approaches, evidence gaps — explore ideas openly before committing to protocol decisions.

Good brainstorm triggers in this context:

- Drafting or revising the PICO/SPIDER framework for the research question
- Generating database search strings (ACM DL, IEEE Xplore, Scopus, Web of Science, arXiv)
- Deciding inclusion/exclusion criteria for title/abstract vs. full-text screening
- Mapping evidence themes and identifying gaps before writing the synthesis

## Conventions for methodology notes

- One MD file per topic/decision (e.g., `methodology/search-strategy.md`, `methodology/inclusion-exclusion.md`)
- Lead each file with a one-paragraph rationale for the decision, then details
- Date significant decisions in the frontmatter or first heading (ISO 8601: `2026-06-07`)
- Record rejected alternatives briefly — future reviewers need to know why a path was not taken

## Scripts

All scripts used during the SLR live in `slr/scripts/` — treat this directory as the project root's `scripts/` folder for the SLR project. That keeps tooling co-located with the work that needs it.

### Header block

Every script must open with a comment block covering:

1. What the script does
2. Its inputs and outputs (file paths, flags, environment variables)
3. Any dependencies or environment requirements

### Parameterisation

If a script is likely to be run more than once or by more than one person, **parameterise it** rather than hard-coding values:

- **Shell scripts**: use `--flag value` long-form options (see `archive-session.sh` for the pattern)
- **Python scripts**: use `argparse` subcommands (see `zotero_client.py` for the pattern)
- Avoid positional-only arguments for anything non-obvious — flags are self-documenting
- Include a `--help` / `-h` path that prints usage

One-off throwaway scripts are exempt; if in doubt, parameterise.

### Existing scripts

| Script | Purpose |
| --- | --- |
| `archive-session.sh` | Create timestamped checkpoint or summary in `methodology/chat-history/` |
| `auto-archive-hook.sh` | Claude Code Stop hook — appends turn markers to the daily rolling log |
| `zotero_client.py` | CLI wrapper for common Zotero API operations (search, tag, list) |
| `install-search-tools.sh` | Bootstrap Exa, Semantic Scholar, and Zotero on a new machine |

---

## Zotero integration

The SLR uses a Zotero library to manage references through screening and synthesis stages.

### Setup

1. Copy `slr/.env.example` to `slr/.env` and fill in your keys.
2. Create two API keys at <https://www.zotero.org/settings/keys>:
   - **Read-only key**: scoped to the SLR library; used for search, listing, and reading items. Set `ZOTERO_API_KEY_READONLY`.
   - **Read-write key**: scoped to the SLR library; used for adding tags and updating items. Set `ZOTERO_API_KEY_READWRITE`. Use sparingly — write operations are irreversible in Zotero without a backup.
3. Set `ZOTERO_GROUP_ID` to your shared group library ID (visible in the library URL) and `ZOTERO_LIBRARY_TYPE=group`.

The `.env` file is gitignored. Never commit real API keys.

### Using the client

```bash
# List all collections
python slr/scripts/zotero_client.py list-collections

# Search for items (readonly)
python slr/scripts/zotero_client.py search --query "generative AI metadata" --brief

# Add a screening tag to an item (uses read-write key)
python slr/scripts/zotero_client.py add-tag --key ABC123XY --tag "screened-include"

# List items in a collection with brief output
python slr/scripts/zotero_client.py list-items --collection COLLKEY --brief
```

### Tag conventions for the SLR

Use consistent tags for each screening phase so items can be filtered by status:

| Tag | Meaning |
| --- | --- |
| `screened-include` | Passed title/abstract screen |
| `screened-exclude` | Failed title/abstract screen |
| `fulltext-include` | Passed full-text screen |
| `fulltext-exclude` | Failed full-text screen |
| `synthesis-included` | In the final synthesis |
| `duplicate` | Duplicate; excluded after dedup |

---

## Related resources

- Nitrate Online corpus: [`ScottThurlow/nitrateonline`](https://github.com/ScottThurlow/nitrateonline)
- Preferred reporting: [PRISMA 2020](https://www.prisma-statement.org/)
