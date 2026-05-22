# notes/planned-skills.md

Five skills originally listed as `status: "planned"` in `subagentskills/.claude-plugin/marketplace.json`, each sketched at the level of detail an agent would need to do a credible first draft of `SKILL.md`. As of `subagentskills@351f24c7`, the catalog now also includes a sixth authored entry (`anthropic-subprocessor-lanes`, released) — see Authored section at the bottom.

The point of this file is so that whichever session picks up "go author the next skill" doesn't have to re-derive the intent.

---

## 1. `reveal-and-restore-worker-token`

**Pairs with**: nothing — this is an ops pattern, not a plugin driver.

**Trigger phrases**: "rotate the worker token", "I need to read the worker secret", "the worker is using the old token", "update the gh-pr-mcp secret", "reveal and restore", "secret rotation".

**What it does**: encodes the procedure that has been used 5+ times this multi-day session for rotating a Cloudflare Worker secret binding without leaking the value to chat history or transcripts.

**Sketch of the workflow section**:

```
1. Confirm with the user that the rotation is intentional. NEVER rotate
   on speculation.

2. wrangler secret list                       # show current bindings
3. read the new value into a SCOPED env var:
     read -s NEW_TOKEN                        # -s = silent, no chat echo
4. wrangler secret put GH_TOKEN < <(echo -n "$NEW_TOKEN")
5. unset NEW_TOKEN                            # immediately
6. verify: trigger one no-op gh_request via the worker

If at any point the value would be printed to stdout (e.g. via echo, cat,
curl -v), STOP and recover with `history -c` then `unset NEW_TOKEN`.
```

**Bundled scripts**: probably none — the procedure is short enough to inline. If anything, a `verify.sh` that does a single GET via the worker and checks the response shape.

**References**: `references/cloudflare-api-pattern.md` describing wrangler vs the dashboard vs the API, and when each is appropriate.

---

## 2. `html-effectiveness-builder`

**Pairs with**: `subagenthtml` (the gallery of reference examples).

**Trigger phrases**: "make me an HTML report", "build a self-contained HTML page", "Thariq-format HTML", "I need a clickable HTML diagram", "interactive HTML with no JS dependencies", "the html-effectiveness style".

**What it does**: emits HTML artifacts that match the canon: single file, CSS custom properties theming, mobile-readable, click-to-focus interaction, no JS framework dependencies, no external script tags. The shape of `subagentarch/docs/01-erd-overview.html` is the canonical example.

**Sketch of the workflow section**:

```
1. Decide the artifact type from the user's intent:
   - flowchart           → 13-flowchart-diagram.html shape
   - data dashboard      → 09-slide-deck.html shape
   - status page         → 11-status-report.html shape
   - explainer           → 14-flowchart.html or 15-feature-explainer.html shape
   - planning doc        → 16-implementation-plan.html shape

2. Read the corresponding file from subagenthtml (or anthropics/html-effectiveness)
   to anchor the structure.

3. Pull the user's data into the chosen shape. KEY CONSTRAINTS:
   - one file only
   - CSS custom properties for theming (--clay, --slate, --oat, etc.)
   - viewBox-relative SVG (renders cleanly on any screen)
   - inline <script> at end, NO external src=
   - no Markdown rendering — everything is HTML

4. Validate:
   - opens in mobile Safari without errors
   - all click handlers reachable without console errors
   - prints reasonably to PDF
```

**Bundled assets**: probably `assets/css/base.css` (the design-token set) and `assets/templates/<type>.html` (a stripped scaffold of each artifact type).

**References**: `references/thariq-format-rules.md` with the do/don't list, and `references/css-tokens.md` with the palette and typography.

---

## 3. `subagenttasks-validator`

**Pairs with**: `subagenttasks` package — uses its compiled JSON Schema.

**Trigger phrases**: "validate this tasks.json", "is my tasks file valid", "check my task data", "ajv this", "schema-check the tasks".

**What it does**: drives `ajv-2020` against `subagenttasks/schema/tasks.schema.json` and returns parsed diagnostics with JSON pointers — same shape as `ts_typecheck`'s diagnostics output.

**Sketch of the workflow section**:

```
1. Locate the input file. If user didn't specify, look for tasks.json in cwd.

2. Determine the schema version (subagenttasks pinned major).

3. Run validation:
     npx --no-install ajv validate \
       -s "$(npm root)/@opensubagents/subagenttasks/schema/tasks.schema.json" \
       -d tasks.json --spec=draft2020 --strict=false

4. Parse the output into a structured diagnostics array:
     [{ jsonPointer, keyword, message, params }]

5. If errors: surface each with the offending value AND a suggested fix
   based on the canonical examples in subagenttasks/fixtures/.

6. If clean: confirm and report the schema version used.
```

**Bundled scripts**: `scripts/validate.mjs` — calls ajv programmatically and emits structured JSON (rather than the human-formatted CLI output).

**References**: `references/ajv-error-keys.md` (mapping every ajv error keyword to a human-friendly explanation), `references/fixture-examples.md` (links into `subagenttasks/fixtures/` for "what valid data looks like").

---

## 4. `graphql-erd-from-zod`

**Pairs with**: any repo using Zod for its data model — currently `subagenttasks`, eventually `subagentbriefs`, `subagentflags`, `subagentobs`.

**Trigger phrases**: "diagram my data model", "generate an ERD from these zod schemas", "show me how these types relate", "make a GraphQL schema from my zod", "subagentarch-style ERD".

**What it does**: the path used to bootstrap `subagentarch` itself, captured as a reusable skill. Reads Zod schema files, derives a GraphQL schema as the cross-language source-of-truth, emits an interactive HTML diagram in the Thariq format.

**Sketch of the workflow section**:

```
1. Locate the Zod files. Look for src/schemas/, src/types/, or whatever
   the user names.

2. Extract definitions:
     for each z.object({...}) export named PascalCase →
       a GraphQL `type`
     for each z.enum([...]) →
       a GraphQL `enum`
     for each z.union([...]) of object refs →
       a GraphQL `union`

3. Emit schema/architecture.graphql with header comments tracking
   the Zod source files and timestamps.

4. Lay out entities by cluster (heuristic: prefix-based, e.g. all
   types starting with "Brief" → brief cluster).

5. Emit docs/<NN>-erd-<topic>.html in the Thariq format:
   - one box per type, color-coded by cluster
   - one edge per inter-type reference (solid = single, dashed = list)
   - click-to-focus / dim-others interaction
   - side panel with fields + inbound/outbound refs

6. Validate the round-trip: re-parse the emitted GraphQL via a quick
   graphql-js parse, confirm no syntax errors.
```

**Bundled scripts**: `scripts/zod-to-graphql.mjs` (the AST-walk and emit), `scripts/graphql-to-erd-html.mjs` (the HTML rendering).

**References**: `references/cluster-heuristics.md` (how to decide which entity belongs to which cluster from naming alone), `references/thariq-svg-tricks.md` (curved-path equations for inter-cluster edges).

---

## 5. `brief-author-canonical`

**Pairs with**: `subagentbriefs` package.

**Trigger phrases**: "write a brief", "draft a project brief", "I need a brief for X", "author this in the canonical brief shape", "make this into a brief".

**What it does**: produces a Brief document in the canonical `subagentbriefs` shape: YAML frontmatter (`id`, `title`, `status`, `summary`) + body sections in a fixed order (Context, Outcomes, Approach, Risks, Tasks-to-Emit, Links). Validates against the brief schema before write.

**Sketch of the workflow section**:

```
1. Read the user's seed (could be a paragraph, a bug report, a stakeholder
   email — anything).

2. Map seed → frontmatter:
   - id           kebab-case slug from title
   - title        ≤80 chars
   - status       defaults to "draft"
   - summary      one-paragraph elevator pitch

3. Body sections (in order, ALL required, even if "TBD"):
   ## Context             what is true today; why this matters now
   ## Outcomes            measurable end states
   ## Approach            high-level plan
   ## Risks               what could derail this
   ## Tasks-to-Emit       list of task titles this brief seeds
   ## Links               related issues, PRs, prior briefs

4. If the user gives a partial seed, ask ONE focused clarifying question
   to fill the most-important gap (usually Outcomes). Do NOT bombard
   with a dozen at once.

5. Validate against subagentbriefs/schema/brief.schema.json before save.
```

**Bundled assets**: `assets/templates/brief.md` (the empty starter), `assets/examples/first-brief.md` (real example from `subagentbriefs/briefs/`).

**References**: `references/section-rubrics.md` (what each section should and should NOT contain), `references/canonical-examples.md` (links into `subagentbriefs/briefs/` for "what a finished brief looks like").

---

## Authoring order suggestion

If asked to build them in order, do `reveal-and-restore-worker-token` first (smallest, no dependencies, captures something we already do regularly). Then `subagenttasks-validator` (small, exercises the schema toolchain). Then `brief-author-canonical` (small, mostly templating). `html-effectiveness-builder` and `graphql-erd-from-zod` are bigger and benefit from the smaller three being done first as references.

---

## Authored

### 6. `anthropic-subprocessor-lanes` (released 2026-05-22 in `subagentskills@351f24c7`)

**Pairs with**: an in-repo MCP server at `skills/anthropic-subprocessor-lanes/src/server.ts` (no external sibling plugin; the server is bundled with the skill).

**Trigger phrases**: "who does Anthropic use as a subprocessor", "list Anthropic's vendors", "trust.anthropic.com lanes", "npm + github + skills.sh for X", "subprocessor compliance report", "supply chain snapshot Anthropic".

**What it does**: joins the 18-entry list at `trust.anthropic.com/subprocessors` against three public developer-surface lanes (npm scope, GitHub org, skills.sh page). The trust page is Vanta-rendered so the list is hand-maintained in `data/subprocessors.json`; per-lane snapshots were verified 2026-05-21 by scraping each lane from a Cloudflare container (the chat sandbox's egress proxy blocks npmjs.com / github.com / skills.sh).

**Three tools**:
- `anthropic_subprocessors` — read the snapshot, zero network
- `anthropic_subprocessor_report` — live fan-out across all 18×3 lanes
- `org_lanes` — ad-hoc lookup for any single org (not just Anthropic's)

**Bundled assets**: `data/subprocessors.json` (the canonical list + verified snapshot), `references/lane-quirks.md` (Cloudflare/Twilio unscoped, Sentry skills.sh 404, ElevenLabs npm WAF, GitHub rate limits).

**Why this was authored ahead of the order above**: emerged from a real session that joined the trust page against the user's prior interest in Cloudflare and Anthropic's developer surfaces. The session produced both the data and the code, so it shipped as a finished skill rather than a sketch.
