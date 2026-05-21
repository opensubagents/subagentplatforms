# submodules/

This directory holds the 8 canonical `opensubagents/*` repos as git submodules. It is intentionally empty in the bootstrap commit — the gitlink tree entries (mode `160000`) that make submodules work cannot be written via GitHub's Contents API, which is the only write path available to the worker that bootstrapped this repo.

## To materialize the submodules

Clone this repo locally and run:

```bash
git clone https://github.com/opensubagents/subagentplatforms.git
cd subagentplatforms

git submodule add https://github.com/opensubagents/ts-bootstrap-mcp.git    submodules/ts-bootstrap-mcp
git submodule add https://github.com/opensubagents/subagentskills.git      submodules/subagentskills
git submodule add https://github.com/opensubagents/subagentarch.git        submodules/subagentarch
git submodule add https://github.com/opensubagents/subagenttasks.git       submodules/subagenttasks
git submodule add https://github.com/opensubagents/subagenttaskmaster.git  submodules/subagenttaskmaster
git submodule add https://github.com/opensubagents/subagentbriefs.git      submodules/subagentbriefs
git submodule add https://github.com/opensubagents/subagenthtml.git        submodules/subagenthtml
git submodule add https://github.com/opensubagents/subagentlsp.git         submodules/subagentlsp

git commit -m "chore: materialize submodules"
git push
```

Once that commit lands, any subsequent clone with `git clone --recurse-submodules` will check out every opensubagents/* repo into this directory at its pinned SHA.

## Why pin SHAs vs follow `main`?

`.gitmodules` declares `branch = main` for every submodule, so `git submodule update --remote` fast-forwards each one to its current `main`. The default `git submodule update` honors the SHA recorded at the time of the meta-repo's last commit, giving you a reproducible snapshot.

## Suggested workflow

- Treat the meta-repo as the canonical "current state of the org" snapshot.
- Bump submodule SHAs in a single meta-commit when a checkpoint is worth preserving (release, demo, retro).
- Day-to-day work happens in the individual submodule repos, not here.
