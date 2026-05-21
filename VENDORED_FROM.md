# VENDORED_FROM

This meta-repo does **not** vendor any third-party code. Several upstreams are read for reference during work in the sandbox (`/home/claude/`) and shape the patterns used here, but their content stays at the source.

## Read-for-reference (cloned during sandbox sessions, never republished)

| Upstream | What we read it for | License |
|---|---|---|
| [microsoft/vscode](https://github.com/microsoft/vscode) → `extensions/html-language-features/server` | LSP wrapper around the HTML language service | MIT |
| [microsoft/vscode-html-languageservice](https://github.com/microsoft/vscode-html-languageservice) | The HTML parser + LSP services the wrapper calls | MIT |
| [anthropics/swift-markdown](https://github.com/anthropics/swift-markdown) | Markdown parser used in claude.ai consumer apps (Anthropic's fork of swiftlang/swift-markdown) | Apache-2.0 |
| [anthropics/swift-markdown-ui](https://github.com/anthropics/swift-markdown-ui) | SwiftUI Markdown renderer (fork) — what the iOS app uses | MIT |
| [anthropics/html-effectiveness](https://github.com/anthropics/html-effectiveness) | The 20-file HTML gallery referenced in the Sep 2025 "HTML is unreasonably effective" post | Custom (see repo) |
| [anthropics/skills](https://github.com/anthropics/skills) | The skill-creator tooling: `quick_validate.py`, `package_skill.py`, the SKILL.md spec | MIT |
| [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) | Marketplace.json shape with SHA-pinned source entries | Apache-2.0 |
| [cloudflare/claude-managed-agents](https://github.com/cloudflare/claude-managed-agents) | The managed-agents primitives (Session, Vault, Dream, etc) we model in `subagentarch` | Apache-2.0 |
| [cloudflare/miniflare](https://github.com/cloudflare/miniflare) | Local Workers runtime for testing the `gh-pr-mcp` worker | MIT |
| [cloudflare/skills](https://github.com/cloudflare/skills) | Cloudflare's published skills — pattern reference | Apache-2.0 |
| [cloudflare/workers-mcp](https://github.com/cloudflare/workers-mcp) | MCP-over-Workers pattern, basis of `gh-pr-mcp` | Apache-2.0 |
| [ChromeDevTools/chrome-devtools-mcp](https://github.com/ChromeDevTools/chrome-devtools-mcp) | DevTools-protocol MCP server, installed globally in the sandbox | Apache-2.0 |
| [eyaltoledano/claude-task-master](https://github.com/eyaltoledano/claude-task-master) | Upstream that `subagenttaskmaster` rebrands | MIT |
| [agentskills.io](https://agentskills.io/specification) | The Agent Skills specification — followed by `subagentskills` and the `ts-bootstrap` skill | (spec, not code) |

## How to refresh references

```bash
# Read-only mirror into the sandbox for inspection. Never commit these.
for repo in microsoft/vscode-html-languageservice anthropics/swift-markdown ...; do
  git clone --depth 1 "https://github.com/$repo" "/home/claude/$(basename $repo)"
done
```

If an upstream introduces a breaking change that affects a sibling opensubagents/* repo, update the affected repo and bump its submodule SHA here.
