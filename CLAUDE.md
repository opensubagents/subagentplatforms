# CLAUDE.md

Workspace-level guidance for Claude (and any other coding agent) working inside `opensubagents/subagentplatforms`.

## What this repo is

A **meta-workspace** — the single-clone entry point to the entire opensubagents stack. It contains:

- A map of all 8 sibling repos (README.md)
- Curated cross-cutting notes that don't belong inside any one sibling (notes/)
- An interactive HTML diagram of the org (diagrams/org-overview.html)
- Submodule wiring (.gitmodules + submodules/)

It does **not** contain any source code, plugins, or skills. Those live in their canonical repos and are referenced via submodules.

## What to do when asked to "update the workspace"

1. If a sibling repo got a meaningful commit worth pinning, update its submodule SHA:

   ```bash
   cd submodules/<repo>
   git fetch origin && git checkout origin/main
   cd ../..
   git add submodules/<repo>
   git commit -m "chore(submodules): bump <repo> to $(cd submodules/<repo> && git rev-parse --short HEAD)"
   ```

2. If a cross-cutting note (architecture map, journal, planned-skills list) is stale, edit the relevant file in `notes/` directly.

3. If a new sibling repo joins the org, add it to:
   - `.gitmodules`
   - `submodules/README.md` (the materialization command list)
   - The README.md "Sibling repos" table
   - `diagrams/org-overview.html` (the visualization)

## What NOT to do here

- **Do not commit source code.** Every executable artifact lives in a sibling repo. If you find yourself wanting to add a `src/` directory, you're in the wrong repo — open a PR on the relevant sibling instead.
- **Do not duplicate content from sibling repos.** Reference, don't copy. If the same fact lives in two places, one of them will rot.
- **Do not commit anything from `/mnt/transcripts/`, `/mnt/user-data/uploads/`, or `/mnt/skills/`** if you're working from a sandbox. Those are read-only, contain sensitive context, and are not yours to publish.

## Build artifacts

There are none in this repo. There is nothing to build.

## Sibling repos that drive everything

```
@opensubagents/ts-bootstrap-mcp     MCP plugin: scaffold + install + typecheck TS projects
@opensubagents/subagentskills       Agent Skills catalog (marketplace.json + skills/)
@opensubagents/subagentarch         GraphQL ERD + Kimball warehouse for the data model
@opensubagents/subagenttasks        Zod schemas + tests for the canonical task data model
@opensubagents/subagenttaskmaster   Rebranded fork of eyaltoledano/claude-task-master
@opensubagents/subagentbriefs       Brief authoring chassis + 323-page reference crawl
@opensubagents/subagenthtml         Private fork of anthropics/html-effectiveness
@opensubagents/subagentlsp          Vendored microsoft/vscode html-language-server
```

See `README.md` for status and `notes/architecture.md` for how they wire together.

## House style for notes

- Markdown. No JSX, no Mermaid in markdown (use `diagrams/*.html` for visuals).
- Tables for cross-repo state. Prose for everything else.
- Curate ruthlessly: a note that hasn't been read in 3 months should be deleted, not preserved.
- Cite SHAs when referencing a sibling repo's state: ``in `subagentarch@15228221`'' beats "in subagentarch".
