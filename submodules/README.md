# submodules/

This directory holds the 8 canonical `opensubagents/*` repos as git submodules, materialized at commit `45e6b2a7`.

## Use

```bash
git clone --recurse-submodules https://github.com/opensubagents/subagentplatforms.git
```

That single command gives you the meta-repo plus every sibling checked out at the SHA pinned in this repo's HEAD.

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Pinned submodules

| Path | URL | Pinned SHA |
|---|---|---|
| `submodules/ts-bootstrap-mcp` | `github.com/opensubagents/ts-bootstrap-mcp` | `db22b540` |
| `submodules/subagentskills` | `github.com/opensubagents/subagentskills` | `81ddda98` |
| `submodules/subagentarch` | `github.com/opensubagents/subagentarch` | `15228221` |
| `submodules/subagenttasks` | `github.com/opensubagents/subagenttasks` | `ab329d2c` |
| `submodules/subagenttaskmaster` | `github.com/opensubagents/subagenttaskmaster` | `6a9c632a` |
| `submodules/subagentbriefs` | `github.com/opensubagents/subagentbriefs` | `b538d087` |
| `submodules/subagenthtml` | `github.com/opensubagents/subagenthtml` | `58c305be` |
| `submodules/subagentlsp` | `github.com/opensubagents/subagentlsp` | `fd99391f` |

## How to advance a submodule SHA

When a sibling repo has a new commit worth pinning here:

```bash
cd submodules/<repo>
git fetch origin && git checkout origin/main
cd ../..
git add submodules/<repo>
git commit -m "chore(submodules): bump <repo> to $(cd submodules/<repo> && git rev-parse --short HEAD)"
git push
```

Or, to fast-forward every submodule to its current `main` in one shot:

```bash
git submodule update --remote
git add submodules/
git commit -m "chore(submodules): fast-forward all to main"
git push
```

## Why pin SHAs vs follow `main`?

`.gitmodules` declares `branch = main` for every submodule, so `git submodule update --remote` fast-forwards each one to its current `main`. The default `git submodule update` honors the SHA recorded in this repo's last commit, giving you a reproducible snapshot. Both modes are intentionally available.

## Suggested workflow

- Treat the meta-repo as the canonical "current state of the org" snapshot.
- Bump submodule SHAs in a single meta-commit when a checkpoint is worth preserving (release, demo, retro).
- Day-to-day work happens in the individual submodule repos, not here.

## Materialization history

The initial bootstrap commit (`14364fe5` and earlier) left this directory empty because `/git/trees` write access wasn't yet propagated on the freshly-created repo. Commit `45e6b2a7` materialized all 8 gitlinks directly via the Git Data API once the access propagated, without requiring a local clone.
