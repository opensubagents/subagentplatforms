# Planned skills

The 5 placeholders in `subagentskills/.claude-plugin/marketplace.json` with `status: "planned"`. Each has a one-paragraph sketch of what to build when it gets authored. None of these have `SKILL.md` files yet — adding them is the work.

## 1. reveal-and-restore-worker-token

**Purpose.** Codify the operational pattern for rotating a Cloudflare Worker secret in place without leaking it to chat history. Used 5+ times across the multi-day session.

**Trigger phrases.** "rotate the worker token", "reveal the secret", "I need to read the API key from the worker", "swap the binding".

**Workflow.**
1. `wrangler secret list` — confirm the secret name exists
2. `wrangler secret put <NAME>` with a sentinel that includes the worker's own request to print-and-restore. The worker reads the sentinel, captures the prior value into a scoped env var, prints it once into a constrained channel (NEVER chat), then restores the binding.
3. The agent never sees the secret in chat history; the operator captures it via the worker's response only.

**Why not just `wrangler secret get`.** Cloudflare does not expose secret reads; the worker must reveal itself, and only once, on a one-shot path that's deleted after use.

**Pairs with.** No new MCP plugin — uses Wrangler directly.

## 2. html-effectiveness-builder

**Purpose.** Generate self-contained HTML artifacts matching the Thariq html-effectiveness gallery format. Used to produce `subagentarch/docs/01-erd-overview.html` and the org-overview diagram in this repo.

**Trigger phrases.** "build me an HTML report", "make a Thariq-style diagram", "render this data as a one-page HTML", "I want an interactive viewer", "Thariq format".

**Format invariants.**
- CSS custom properties for theming (the ivory/clay/slate/oat palette from anthropics/html-effectiveness)
- No JS dependencies beyond inline; opens in Safari on iPhone
- Hover/click interactions via tiny vanilla JS at the bottom of the file
- One file, one URL, one PR

**Pairs with.** No MCP plugin — pure file authoring. Bundles a `references/style-system.md` extracted from the anthropics/html-effectiveness gallery.

## 3. subagenttasks-validator

**Purpose.** Validate a `tasks.json` file against the canonical `@opensubagents/subagenttasks` JSON Schema using ajv-2020.

**Trigger phrases.** "is my tasks.json valid", "does this match the task schema", "validate this against subagenttasks", "what's wrong with my task data".

**Workflow.**
1. Read the schema from `subagenttasks/schemas/tasks.schema.json`
2. Compile with `ajv-2020` (NOT ajv default — schema is draft 2020-12)
3. Validate the user's input file
4. Return parsed diagnostics with JSON Pointers, line numbers if possible

**Pairs with.** `@opensubagents/subagenttasks` (consumed as the schema source). Bundles a `scripts/validate.mjs` that handles the ajv-2020 compile + validate dance.

## 4. graphql-erd-from-zod

**Purpose.** Derive a GraphQL ERD from a Zod schema file, then emit a Thariq-format interactive HTML diagram. This is the path that bootstrapped `subagentarch` and should be repeatable.

**Trigger phrases.** "turn my zod schemas into a GraphQL ERD", "I want a diagram of my types", "generate an ERD from this file", "visualize my schema".

**Workflow.**
1. Parse the Zod source file with `zod-to-json-schema` or by introspection
2. Map Zod types to GraphQL: `z.string()` → `String!`, `z.array(X)` → `[X!]!`, `z.union([A, B])` → GraphQL union, etc.
3. Detect references (one type containing another) and emit them as ERD edges
4. Render via the html-effectiveness-builder skill (composition)

**Pairs with.** Composes with `html-effectiveness-builder`. No new MCP plugin — bundles a `scripts/zod-to-erd.mjs`.

## 5. brief-author-canonical

**Purpose.** Author a brief in the canonical `subagentbriefs` shape (frontmatter + summary + body + initiatives + tasks-to-emit links). Validates against the brief schema before write.

**Trigger phrases.** "write a brief about X", "I need a new brief", "create a project brief", "draft a brief in our format".

**Workflow.**
1. Interview the user for: title, summary (1–2 sentences), problem statement, success criteria, stakeholders, related links
2. Read the brief schema from `subagentbriefs/schemas/brief.schema.json`
3. Emit the brief with the canonical frontmatter shape
4. Optionally generate the initial task list (tasks-to-emit) following the `subagenttasks` Task shape
5. Validate before writing

**Pairs with.** `@opensubagents/subagentbriefs` (schema source) and indirectly `@opensubagents/subagenttasks` (for the tasks-to-emit). Bundles `references/brief-shape.md` and `assets/template-brief.md`.

## How to promote a sketch to a real skill

1. Read this file's section for the skill.
2. Copy `subagentskills/template/SKILL.md` to `subagentskills/skills/<name>/SKILL.md`.
3. Author the SKILL.md following the sketch. Keep `description` ≤ 1024 chars and avoid `<` `>` characters.
4. Validate with `quick_validate.py` from `anthropics/skills/skill-creator/scripts/`.
5. Flip the marketplace.json entry's `status` from `"planned"` to `"released"` and bump its `version` from `"0.0.0"` to `"0.1.0"`.
6. Add a journal entry here noting the promotion.
