# Architecture — how the 8 repos compose

The opensubagents stack is layered by **what is owned where**, not by what runs where. This document maps the ownership boundaries and the data flow that crosses them.

## Layers

```
Layer 4  PLATFORMS                        subagentplatforms (this repo)
         ──────────────                    cross-cutting docs, pinned manifest,
                                           journal, architecture overview

Layer 3  ORCHESTRATION + CATALOGS         subagentskills           ts-bootstrap-mcp
         ─────────────────────────         marketplace.json,        7 MCP tools,
                                           paired skill content      streamable HTTP
                                                                     or stdio

Layer 2  DATA MODELS + REFERENCES         subagenttasks            subagentbriefs
         ─────────────────────────         schemas/, types/,        PRD, brief shape,
                                           fixtures/, tests/         first brief,
                                                                     323p Hamster crawl

                                          subagentarch              subagentlsp
                                          GraphQL ERD,              vendored MS
                                          warehouse DDL,             vscode html LSP
                                          interactive HTML

Layer 1  RENDERING + VENDORING            subagenthtml (private)    subagenttaskmaster
         ─────────────────────────         fork of anthropics/      vendored
                                           html-effectiveness        eyaltoledano/
                                                                     claude-task-master
```

A change in Layer 2 (e.g., a new field in `subagenttasks/schemas/tasks.schema.json`) propagates upward: Layer 3 catalogs add a validator skill or a new MCP tool, Layer 4 bumps the SHA in `manifest.json` and notes the change in `journal.md`. Layer 1 rarely moves; it's the foundation.

## Data flow during a typical session

```
operator question                       Claude session
                ↓                              ↓
      reads subagentplatforms          reads cached skill descriptions
      to pick the right repo           from /mnt/skills/ + ~/.claude/skills/
                ↓                              ↓
                └──────────────┬───────────────┘
                               ↓
                  triggers a skill (e.g. ts-bootstrap)
                               ↓
                  skill orchestrates MCP plugin tool calls
                  (ts-bootstrap-mcp's 7 tools)
                               ↓
                  plugin shells out to npm/tsc/tsx
                  in the cloud sandbox
                               ↓
                  output flows back through MCP
                  protocol → skill → session
                               ↓
                  on a successful outcome, commits
                  may be pushed via the gh-pr-mcp worker
                  to the appropriate sibling repo
```

## Ownership boundaries

| Concern | Owner |
|---|---|
| Canonical task schema (a `Task` is …) | `subagenttasks` |
| Canonical brief shape (a `Brief` is …) | `subagentbriefs` |
| Canonical agent-runtime primitives (Agent, Session, Vault…) | upstream: `cloudflare/claude-managed-agents`; we model in `subagentarch` |
| Skill format (frontmatter, `.skill` zip) | upstream: agentskills.io; we follow in `subagentskills` |
| MCP protocol | upstream: `modelcontextprotocol/typescript-sdk`; we implement in `ts-bootstrap-mcp` |
| Cross-repo notes, journal, manifest | here (`subagentplatforms`) |

When upstream churns (e.g., `@modelcontextprotocol/sdk` ships a breaking 2.x), the plugin authors track it in `ts-bootstrap-mcp/VENDORED_FROM.md` and bump the dep in the plugin's own `package.json`. The change ripples to here via a SHA bump in `manifest.json` and a journal entry.

## What runs where

| Component | Runtime |
|---|---|
| MCP plugin (`ts-bootstrap-mcp`) | Node ≥18 in a cloud sandbox (claude.ai web/mobile chat, Claude Code, etc.) over stdio; optionally streamable HTTP on a Cloudflare Worker |
| `gh-pr-mcp` worker | Cloudflare Worker (long-lived, dual-token model) |
| Skills | Loaded into the agent's context window by the host (claude.ai loads from `/mnt/skills/`; Claude Code from `~/.claude/skills/`) |
| Warehouse DDL (`subagentarch/warehouse/`) | AlloyDB or stock Postgres ≥17, currently as-authored — not executed against a live instance yet |
| Interactive HTML diagrams (`subagentarch/docs/*.html`) | Static, opens in any browser; no server, no JS deps beyond inline |

## Future migrations

When the gh-pr-mcp worker token gains `git/trees: write` access to newly-created repos:

1. Replace `manifest.json` with `.gitmodules`. The submodule entries need `160000` tree mode, which only the Trees API can write.
2. Add `.github/workflows/validate-frontmatter.yml` and `validate-marketplace.yml` to `subagentskills` (currently provided as downloadable artifacts but not committed).
3. Re-push `skills/ts-bootstrap/scripts/{check,install}.sh` with mode `100755` via the Trees API — currently `100644` due to Contents API limitation.

When publishing to npm becomes appropriate:

1. `ts-bootstrap-mcp` → `@opensubagents/ts-bootstrap-mcp` (requires npm org claim).
2. `subagenttasks` → `@opensubagents/subagenttasks`.

Neither is blocking today; `npx -y github:opensubagents/...` covers consumption in cloud sandboxes.

## Cross-cutting packages still in design

Listed in `subagentarch/schema/architecture.graphql` as G4 entities but not yet shipped:

- **subagentflags** — OpenFeature flags backed by Cloudflare KV. Adds `FeatureFlag`, `FlagVariant`, `Targeting` to the ERD.
- **subagentobs** — OpenTelemetry tracing with `gen_ai.*` semconv attributes, sunk into Cloudflare Analytics. Adds `Trace`, `Span` to the ERD.

These are placeholders in the schema until a real consumer exists.
