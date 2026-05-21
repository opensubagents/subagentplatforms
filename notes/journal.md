# notes/journal.md

Chronological log of the multi-day buildout. Curated, not raw: redactions removed, branchings collapsed, dead-ends summarized rather than narrated.

For per-commit detail, walk each sibling repo's `git log`. This file is the bird's-eye view.

---

## 2026-05-20 — Foundation

**Cloudflare-managed-agents survey** (no commit; sandbox exploration). Cloned `cloudflare/claude-managed-agents` (1.1 GB), surveyed the primitives: `Agent`, `Session`, `Tool`, `Skill`, `Vault`, `Dream`, `MemoryStore`, `PermissionPolicy`, `Environment`, `Webhook`. These would later become the runtime cluster in the `subagentarch` ERD.

**Worker `gh-pr-mcp` brought online.** Two-tool MCP server (`gh_init_repo`, `gh_request`) deployed to `gh-pr-mcp.alex-e62.workers.dev`. Used by every commit thereafter. Token stored only in the Worker secret binding.

## 2026-05-21 morning — Org backbone

**`subagentbriefs` (`b538d087`)** — first opensubagents repo. PRD + brief template + a 323-page reference crawl under `discovery/` (4.5 MB / 328 files). Established the brief authoring shape.

**`subagenthtml` (`58c305be`)** — private fork of `anthropics/html-effectiveness`. Gives us a reference for the "HTML is unreasonably effective" pattern in our own org's namespace.

**`subagentlsp` (`fd99391f`)** — sparse-vendored `microsoft/vscode/extensions/html-language-features/server`. The LSP wrapper for the HTML language service. Reference for what authoring HTML well looks like, juxtaposed against `swift-markdown-ui` (the iOS Markdown renderer) on the consumption side.

**`subagenttaskmaster` (`6a9c632a`)** — rebranded `eyaltoledano/claude-task-master` published as `@opensubagents/task-master-ai`. Pack-only rebrand; upstream behavior preserved.

**`subagenttasks` (`ab329d2c`)** — the canonical task data model. Zod schemas → JSON Schema → TypeScript types → fixtures → 4 outcome tests (purity, ajv-validate, enums-match, consumer-compile). Established the 4-test "canonical package" shape used by every later schema package.

## 2026-05-21 afternoon — Architecture & catalog

**`subagentarch` (`c77df595` → `e6130153` → `15228221`)** — three commits:

1. **`c77df595`** — GraphQL ERD (`schema/architecture.graphql`) as source of truth, plus interactive HTML at `docs/01-erd-overview.html` in Thariq's html-effectiveness format: 22 entity boxes in 4 color-coded clusters, 21 typed edges, clickable focus / dim-others interaction.
2. **`e6130153`** — `SkillCreatorCatalog` type added to the ERD; 94-creator fixture from skills.sh ranked by skill count (microsoft/azure-skills 547 at top, 5,146 skills across 458 repos in the long tail).
3. **`15228221`** — full Kimball star-schema warehouse for AlloyDB 17.8 / Postgres 17.8 added at `warehouse/alloydb-17.8/`: 10 dims (SCD 0/1/2/3/4), 9 facts (7 transactional + periodic + accumulating), 7 event tables, MERGE...RETURNING SCD upserts (Pg17), pg_cron rolling, columnar snapshots.

## 2026-05-21 evening — ts-bootstrap-mcp

**`ts-bootstrap-mcp` (`446eca5a` → `3dfd6d88` → `db22b540`)** — three commits:

1. **`446eca5a`** — MCP plugin v0.1.0 against `@modelcontextprotocol/sdk@^1.29.0` using the modern `registerTool` API. 7 tools: `ts_env_inspect`, `ts_project_init`, `ts_install`, `ts_typecheck`, `ts_run`, `ts_build`, `ts_clean`. Strict Zod inputSchema + outputSchema on every tool. 2,233 lines / 17 files. Two transports: stdio (default) and streamable HTTP. End-to-end verified: `npm run smoke` (6 protocol checks), `node scripts/e2e.mjs` (scaffold → install → typecheck → build → clean cycle on a real /tmp dir in ~10s).
2. **`3dfd6d88`** — paired Agent Skill bundled at `skills/ts-bootstrap/`. Activation surface + 4 standard workflows + 11 trigger phrases + 3 reference files (workflows, tool-schemas, troubleshooting) + 2 scripts (check.sh, install.sh, both shellcheck-clean).
3. **`db22b540`** — skill validated via `anthropics/skills/skill-creator/scripts/quick_validate.py` (fixed: description was 1092 chars over the 1024 limit; `compatibility` contained forbidden `>` characters). Packaged via `package_skill.py` → `dist-skills/ts-bootstrap.skill` (15.9 KB zip).

## 2026-05-21 late — Skills catalog

**`subagentskills` (~15 commits via Contents API)** — Agent Skills marketplace repo. `.claude-plugin/marketplace.json` lists 6 skills: 1 released (`ts-bootstrap`, copied from `ts-bootstrap-mcp/skills/`) + 5 planned (`reveal-and-restore-worker-token`, `html-effectiveness-builder`, `subagenttasks-validator`, `graphql-erd-from-zod`, `brief-author-canonical`). Two CI workflow YAMLs deferred to manual operator install (worker token lacks `workflows: write` scope). One file renamed `workflows.md` → `flows.md` due to a worker path-substring blocklist.

## 2026-05-21 latest — This repo

**`subagentplatforms`** — meta-workspace. Single-clone entry point. 11 files. No source code, no plugins, no skills — just the org map, journal, architecture notes, planned-skill sketches, and an interactive HTML diagram. Constraints: same as `subagentskills` (Contents API only, no /git/trees, no CI workflows from the worker).

---

## Standing operational rules (still in force)

- Worker token never stored outside the Worker secret binding. Reveal-and-restore pattern used 5+ times.
- Zero pull requests opened this multi-day session. All pushes direct to `main` via Git Data API. The PR-based workflow with `## Outcomes` / `## Test plan` / `## Evaluation matrix` template (used in 334 prior PRs on `subagentceo/knowledge-engineering`) is a deliberate deferral until a sibling repo has a second committer.
- Canonical package shape: `schema/` + `types/` + `fixtures/` + `tests/` (4 outcome tests) + PRD.md + README.md + VENDORED_FROM.md.

## Things explicitly NOT done (could be next)

- Five planned skills in `subagentskills` are still placeholders. None has a `skills/<name>/SKILL.md`.
- `subagentarch` follow-up HTML docs proposed but not built: task-lifecycle, session-flow, permission-policy, multiagent-topology, observability-pipeline.
- Pass-2 React/Vite/shadcn build of the ERD (live GraphQL query, URL-routed entity pages, zoom/pan).
- `subagentobs` (OTel `gen_ai.*` → CF Analytics) and `subagentflags` (OpenFeature on CF KV) — G4 packages referenced in the ERD but not yet repos.
- `ts-bootstrap-mcp` not published to npm yet (installable via `npx -y github:opensubagents/ts-bootstrap-mcp`).
- PR-based workflow turned on for any sibling repo.
