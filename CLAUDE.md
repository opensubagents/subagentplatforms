# CLAUDE.md

Workspace-level guidance for Claude (and other coding agents) operating across the opensubagents stack from this meta-repo.

## What this repo is

A pinned snapshot of the 8 sibling repos + the cross-cutting docs nobody else owns. It is **not** a source-of-truth for any code or schema — every code change still happens in the canonical sibling repo. This repo's job is composition: knowing which SHAs of which repos work together, and recording the decisions that span them.

## Where to make a change

| Kind of change | Repo to edit |
|---|---|
| Task data model (schemas, types, fixtures) | `subagenttasks` |
| Brief shape or first-brief content | `subagentbriefs` |
| Architecture diagrams or warehouse DDL | `subagentarch` |
| New MCP tool for TS bootstrap | `ts-bootstrap-mcp` |
| New skill catalog entry | `subagentskills` |
| Cross-repo notes, journal, planned-skills sketches | **this repo** (`subagentplatforms`) |
| Bump the pinned SHA of any sibling | `manifest.json` in this repo |

**Rule of thumb:** if your change references concepts from more than one sibling repo, it probably lives here. If it touches one repo's internals, it belongs there.

## Standing operational rules (carry over from prior sessions)

1. **Worker token is never stored anywhere except the worker's secret binding.** Reveal-and-restore is the canonical rotation pattern; do not commit a revealed token to chat history or any repo.
2. **Direct push to `main` via the gh-pr-mcp worker is the norm** for opensubagents repos. PRs are not used for solo work; they may be added later when the project takes contributors.
3. **Trust the operator's goal; verify the operator's ingredients.** When the operator names a library or stack, look it up before treating it as a constraint.
4. **Canonical package shape:** `schema/ + types/ + fixtures/ + tests/ + PRD.md + README.md + VENDORED_FROM.md`. Used in `subagenttasks` and `subagentbriefs`.
5. **Decompose into linear outcomes; execute.** Don't ask permission for a step that's clearly required by the stated goal.

## Don't commit from here

This meta-repo lives next to a sandbox that has read-only mounts and ephemeral working dirs. Things that are NEVER safe to commit:

- `/mnt/skills/*` — Anthropic property, read-only by license
- `/mnt/transcripts/*` — prior session transcripts; may contain reveal-and-restore content and other operator-private context
- `/mnt/user-data/uploads/*` — operator's uploaded screenshots, configs, READMEs
- `/mnt/project/*` — project-attached files (Claude.ai feature)
- `node_modules/`, `dist/`, `.cache/`, `__pycache__/` — regeneratable build artifacts
- Vendor clones in `/home/claude/` from Microsoft, Anthropic, Cloudflare, ChromeDevTools — see `VENDORED_FROM.md` for the list

## Agent etiquette across the stack

- When in doubt about a sibling repo's intent, read its `README.md` and its `CLAUDE.md` (if present) BEFORE proposing changes.
- When bumping a SHA in `manifest.json`, leave a one-line note in `notes/journal.md` explaining what changed in the sibling repo.
- When deciding between writing notes here vs. authoring a new skill in `subagentskills`, prefer the skill — it activates contextually for future sessions. Notes here are passive reference.
