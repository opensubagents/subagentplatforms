# subagentplatforms

> Workspace meta-repo for the opensubagents stack. Pins all 8 sibling repos at specific SHAs, plus cross-cutting docs nobody else owns.

## The stack at a glance

| Repo | Role | SHA |
|---|---|---|
| [`subagentbriefs`](https://github.com/opensubagents/subagentbriefs) | PRD + brief shape + first canonical brief | `b538d087` |
| [`subagenttasks`](https://github.com/opensubagents/subagenttasks) | task data model: Zod schemas, JSON Schema, types, tests | `ab329d2c` |
| [`subagenttaskmaster`](https://github.com/opensubagents/subagenttaskmaster) | vendored claude-task-master + rebrand pack | `6a9c632a` |
| [`subagentarch`](https://github.com/opensubagents/subagentarch) | GraphQL ERD + interactive HTML + Kimball warehouse DDL | `15228221` |
| [`subagenthtml`](https://github.com/opensubagents/subagenthtml) (private) | fork of anthropics/html-effectiveness | `58c305be` |
| [`subagentlsp`](https://github.com/opensubagents/subagentlsp) | vendored microsoft/vscode html-language-server | `fd99391f` |
| [`ts-bootstrap-mcp`](https://github.com/opensubagents/ts-bootstrap-mcp) | MCP plugin: 7 tools for TS toolchain bootstrap | `db22b540` |
| [`subagentskills`](https://github.com/opensubagents/subagentskills) | catalog of Agent Skills (1 released, 5 planned) | `17557d5f` |

External: `gh-pr-mcp` Cloudflare Worker at `gh-pr-mcp.alex-e62.workers.dev` v0.4.1.

## Clone the whole stack

The canonical reference is [`manifest.json`](./manifest.json). Use it to pin everything to known-good SHAs:

```bash
mkdir opensubagents && cd opensubagents
curl -sSL https://raw.githubusercontent.com/opensubagents/subagentplatforms/main/manifest.json \
  | python3 -c '
import json, subprocess, sys
m = json.load(sys.stdin)
for r in m["repos"]:
    if r.get("visibility") == "private":
        print(f"# {r[\"name\"]} is private — skip or auth first", file=sys.stderr); continue
    subprocess.run(["git","clone","--depth","1",r["url"]])
    subprocess.run(["git","-C",r["name"],"checkout",r["sha"]])
'
```

When the worker's Git Trees API access is unblocked (see [Known constraints](#known-constraints)), this repo can be reshaped to use proper `.gitmodules` and `git clone --recurse-submodules` instead.

## Cross-cutting docs (the part that lives ONLY here)

- [`notes/journal.md`](./notes/journal.md) — chronological session journal across the multi-day buildout
- [`notes/architecture.md`](./notes/architecture.md) — how the 8 repos compose: data flow, ownership boundaries, what runs where
- [`notes/planned-skills.md`](./notes/planned-skills.md) — sketches for the 5 placeholder skills in `subagentskills/marketplace.json`
- [`diagrams/org-overview.html`](./diagrams/org-overview.html) — Thariq-format interactive map of the 8 repos
- [`VENDORED_FROM.md`](./VENDORED_FROM.md) — vendor clones we read from but never republish (Microsoft, Anthropic, Cloudflare, ChromeDevTools)

## Known constraints

The `gh-pr-mcp` worker token currently has:

- `contents: write` ✓
- `git/trees: write` ✗ — blocked on freshly-created repos (403)
- `workflows: write` ✗ — blocked on `.github/workflows/*` AND on any path containing `workflows`
- Contents API used as fallback **cannot set mode 100755** on executable files

These shaped some decisions in this repo: no `.gitmodules` yet (uses `manifest.json` instead), no `.github/workflows/*` (this repo doesn't need CI of its own — it carries docs, not validated content). When the constraints are lifted, see the migration notes in [`notes/architecture.md`](./notes/architecture.md#future-migrations).

## License

MIT.
