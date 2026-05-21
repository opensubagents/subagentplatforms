# VENDORED_FROM

Vendor source trees we read from during opensubagents work but **do not republish**. These remain at their upstreams; we treat the URLs as reference.

## Reference clones used across sessions

| Upstream | Why we read it | Notes |
|---|---|---|
| [microsoft/vscode](https://github.com/microsoft/vscode) (sparse: `extensions/html-language-features/server/`) | LSP wrapper around the HTML language service; reference implementation we vendor in `subagentlsp` | sparse-cloned, 48 MB |
| [microsoft/vscode-html-languageservice](https://github.com/microsoft/vscode-html-languageservice) | Actual HTML parser + LSP service engine; the layer below the wrapper | 1.7 MB |
| [anthropics/html-effectiveness](https://github.com/anthropics/html-effectiveness) | Gallery of HTML report patterns; our `subagenthtml` is a private fork | 744 KB |
| [anthropics/swift-markdown](https://github.com/anthropics/swift-markdown) | Anthropic fork of the Swift Markdown parser; used by the Claude consumer apps | 2.1 MB |
| [anthropics/swift-markdown-ui](https://github.com/anthropics/swift-markdown-ui) | SwiftUI Markdown renderer; the layer above swift-markdown | 47 MB |
| [anthropics/skills](https://github.com/anthropics/skills) | Source of the `quick_validate.py` and `package_skill.py` tools we use to validate and ship skills | not cloned locally; fetched on demand |
| [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | 203-entry marketplace.json reference; canonical shape for `subagentskills/.claude-plugin/marketplace.json` | read on demand |
| [cloudflare/claude-managed-agents](https://github.com/cloudflare/claude-managed-agents) | Reference for managed-agents primitives modeled in `subagentarch` (Agent, Session, Vault, MemoryStore, etc.) | 1.1 GB |
| [cloudflare/skills](https://github.com/cloudflare/skills) | Cloudflare's own skill catalog; pattern source for our `.claude-plugin/marketplace.json` | 3.6 MB |
| [cloudflare/miniflare](https://github.com/cloudflare/miniflare) | Reference for local-mode Worker testing patterns | 6.3 MB |
| [cloudflare/workers-mcp](https://github.com/cloudflare/workers-mcp) | Pattern for hosting an MCP server as a Cloudflare Worker (Streamable HTTP transport) | 812 KB |
| [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) | Reference MCP server for browser automation | 17 MB |
| [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) | Upstream task-master we vendored as `subagenttaskmaster` | vendored, not cloned in workspace |
| [agentskills.io/specification](https://agentskills.io/specification) | Authoritative spec for SKILL.md frontmatter, `.skill` zip format, marketplace.json | read on demand |

## Files we run but do not vendor

- `quick_validate.py`, `package_skill.py` from `anthropics/skills/skill-creator/scripts/` — invoked at CI time or interactively, never committed.

## When to update this file

- Whenever a sibling repo starts reading from a new external source — add a row.
- Whenever an upstream changes shape enough that our reading-from assumptions break — note the breakage and the resolution.
