# notes/architecture.md

How the 8 sibling repos wire together. Read after the README; refer to `diagrams/org-overview.html` for the visual version of the same content.

## Three planes

```
┌─────────────────────────────────────────────────────────────────────────┐
│  PLANE 1: data model                                                    │
│  ─ canonical types every other repo agrees on                           │
│                                                                         │
│    subagenttasks       Task / Subtask / Tag / ComplexityReport          │
│    subagentbriefs      Brief / Initiative / Link  (canonical, no fork)  │
│    subagentarch        GraphQL ERD over all of the above PLUS the       │
│                        managed-agents runtime primitives                │
│                        (Session, Agent, Tool, Vault, Dream, ...)        │
└─────────────────────────────────────────────────────────────────────────┘
                              │
                              │  consumed by
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PLANE 2: tooling                                                       │
│  ─ executable code that operates ON the data model                      │
│                                                                         │
│    ts-bootstrap-mcp    MCP plugin: scaffold TS projects, install deps,  │
│                        typecheck, build, clean.                         │
│                        ↳ pairs with the ts-bootstrap skill in           │
│                          subagentskills                                 │
│                                                                         │
│    subagenttaskmaster  Rebranded fork of eyaltoledano/claude-task-master│
│                        Consumes subagenttasks schemas.                  │
│                                                                         │
│    gh-pr-mcp           Cloudflare Worker, 2 tools: gh_init_repo +       │
│       (worker)         gh_request. Every push from a sandbox            │
│                        session goes through here.                       │
└─────────────────────────────────────────────────────────────────────────┘
                              │
                              │  authored using
                              ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  PLANE 3: authoring & presentation                                      │
│  ─ how outputs are shaped for humans                                    │
│                                                                         │
│    subagenthtml        Private fork of anthropics/html-effectiveness.   │
│                        The Thariq-format gallery: clickable,            │
│                        self-contained, CSS-variable-themed HTML.        │
│                                                                         │
│    subagentlsp         Vendored microsoft/vscode HTML LSP server +      │
│                        vscode-html-languageservice. Reference for       │
│                        what authoring HTML well looks like in an        │
│                        editor.                                          │
│                                                                         │
│    subagentskills      Agent Skills catalog. marketplace.json +         │
│                        skills/ts-bootstrap/. Five planned skills        │
│                        (sketches in notes/planned-skills.md).           │
└─────────────────────────────────────────────────────────────────────────┘
```

## Concrete data flows

**1. Author a task** →

```
human writes brief             →  subagentbriefs  ─┐
brief authored against the     →  subagentarch    ├→  feeds into  →  subagenttaskmaster
canonical Brief schema in            (ERD says        which uses                ↓
ERD                                   "Brief points    subagenttasks's       emits tasks.json
                                      at [Task]")     schemas to validate   that subagenttasks
                                                                            schemas validate
```

**2. Bootstrap a new TS project from a sandbox** →

```
operator (in claude.ai session)  →  triggers the ts-bootstrap skill (subagentskills)
                                  →  which directs ts_env_inspect → ts_project_init
                                     → ts_install → ts_typecheck → ts_build
                                  →  served by ts-bootstrap-mcp (the plugin)
                                  →  scaffolds, say, an mcp-server template
                                  →  result: dist/index.js, ready for `claude mcp add`
```

**3. Commit anything from a sandbox** →

```
local edits in /home/claude/<repo>/  →  gh-pr-mcp worker  →  GitHub Git Data API
                                         (the only path,        (Trees on existing
                                          since sandbox can      repos, Contents API
                                          not git push directly) on new repos)
```

## Where each entity lives

Drawn from `subagentarch/schema/architecture.graphql`:

| Entity | Defined in | Lives in (runtime) | Consumed by |
|---|---|---|---|
| `Task`, `Subtask`, `Tag` | `subagenttasks` schema | tasks.json file or a managed-agents `MemoryStore` | `subagenttaskmaster`, agents driving multi-step work |
| `Brief`, `Initiative` | `subagentbriefs` chassis | Markdown files in a project | `subagenttasks` (briefs emit tasks) |
| `Agent`, `Session`, `Tool`, `Skill`, `Vault`, `MemoryStore`, `PermissionPolicy`, `Environment`, `Webhook`, `Dream` | `subagentarch` ERD (mirrors cloudflare/claude-managed-agents) | Managed-agents runtime on `platform.claude.com` | Any host that talks to managed-agents |
| `FeatureFlag` (planned) | `subagentarch` cross-cutting | Cloudflare KV (subagentflags, not yet built) | All sessions |
| `Trace`, `Span` (planned) | `subagentarch` cross-cutting | CF Analytics via OTel (subagentobs, not yet built) | Observability stack |

## What the `gh-pr-mcp` worker actually does

```
POST https://gh-pr-mcp.alex-e62.workers.dev/mcp
  JSON-RPC method: tools/call
  with one of two tools:

    gh_init_repo(owner, name, description, private, auto_init)
       → creates a new repo under github.com/<owner>

    gh_request(method, path, body?)
       → arbitrary GitHub REST call, scoped by the worker's token

  Worker holds a fine-grained PAT in secret binding. Token has:
    ✓ contents: write
    ✗ workflows: write              (blocks .github/workflows/*.yml writes)
    ⚠️ /git/trees write only on     (works on ts-bootstrap-mcp; 403 on
       SOME repos                    fresh subagentskills + subagentplatforms)
    ⚠️ path-substring blocklist on  (blocks `references/workflows.md`)
       "workflows"
```

This is why the bootstrap of every repo in this org so far has happened via Contents API single-file PUTs rather than atomic Trees commits.

## Cross-cutting deferred packages

Both referenced in `subagentarch` ERD as planned, both no repo yet:

- **`subagentflags`** — OpenFeature spec on Cloudflare KV. `FeatureFlag { key, variants[], defaultVariant, targeting }` per the ERD.
- **`subagentobs`** — OpenTelemetry traces with `gen_ai.*` semconv attributes, sink to CF Analytics. `Trace { traceId, session, spans[] }`, `Span { spanId, parentSpanId, name, startTs, endTs, attributes }`.

When either gets a real consumer, spin the repo, add a submodule entry here, and update the ERD.
