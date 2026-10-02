# CLAUDE.md — personal FreeCAD fork

This checkout is a **personal fork** of FreeCAD, used to carry the owner's own
changes while still tracking official FreeCAD. This file exists only on the
`custom` branch; it is not part of upstream FreeCAD.

## Repository layout

| Remote     | URL                                    | Use                                   |
|------------|----------------------------------------|---------------------------------------|
| `origin`   | https://github.com/edemko/FreeCAD      | the fork — push here                  |
| `upstream` | https://github.com/FreeCAD/FreeCAD     | official repo — fetch only            |

- `upstream` push URL is set to `DISABLED` on purpose. Never re-enable it or
  push to FreeCAD/FreeCAD.
- `gh` default repo is `edemko/FreeCAD`, so `gh pr create` targets the fork.

| Branch   | Purpose                                                                 |
|----------|-------------------------------------------------------------------------|
| `custom` | **Default branch** (locally and on GitHub). All personal work goes here. |
| `main`   | Untouched mirror of `upstream/main`. Never commit to it.                |

Feature work: branch off `custom` (`git switch -c my-feature`), merge back
into `custom` when done.

## Syncing official changes — owner merges MANUALLY

The owner merges upstream into `custom` by hand. Do **not** merge upstream
into `custom` on your own initiative; only do it when explicitly asked.

```bash
git fetch upstream
git switch custom
git merge upstream/main      # resolve conflicts, commit
git push
pixi run build               # rebuild after the merge
```

Optionally keep the fork's `main` current (server-side fast-forward, touches
nothing locally): `gh repo sync edemko/FreeCAD -b main`.

A repo-local alias `git sync-upstream` exists in `.git/config` (updates
`main`, then auto-merges it into `custom` and pushes both). It does not match
the owner's manual-merge preference — don't run it unless asked.

## Building and running (macOS arm64, pixi)

The build uses **pixi** (installed via Homebrew). All dependencies live in
`.pixi/` (conda-forge); Homebrew libs are deliberately ignored by the CMake
preset. `build/` and `.pixi/` are gitignored.

```bash
pixi run configure     # first time, or after big upstream changes / CMake errors
pixi run build         # incremental build -> build/relWithDebInfo
pixi run freecad       # launch GUI (build/relWithDebInfo/bin/FreeCAD)
pixi run test          # ctest
```

- If `pixi.lock` changed after a merge, run `pixi install` before configuring.
- Console smoke test:
  `pixi run build/relWithDebInfo/bin/FreeCADCmd -c "import Part; print(Part.makeBox(1,2,3).Volume)"`
- First full build: ~8.3k ninja steps, ~45 min on 10 cores. Build dir ~7 GB,
  `.pixi` ~4 GB. Disk is tight (~45 GB free) — avoid creating extra build
  trees (e.g. debug + release) without checking `df -h` first.
- Python-only changes (many workbenches are Python): `pixi run build` just
  copies files; restart FreeCAD to pick them up.
- The official FreeCAD.app (1.1.3) was removed; this build is the owner's
  FreeCAD. It is launched from the terminal (no .app bundle yet).
- User settings live in `~/Library/Preferences/FreeCAD/` and
  `~/Library/Application Support/FreeCAD/` (versioned subfolders, e.g. `v1-1`).

## Contributing a change back upstream

Read `AI_POLICY.md`. FreeCAD does **not** accept PRs with clearly
AI-generated code, commit messages, PR descriptions or review replies, and
requires disclosure of AI assistance. So:

- Never open PRs, issues or comments against FreeCAD/FreeCAD on the owner's
  behalf.
- If the owner wants to upstream something, prepare it on a separate branch
  created from `upstream/main` (not from `custom`), without this `CLAUDE.md`,
  and let the owner write/submit the PR themselves
  (`gh pr create --repo FreeCAD/FreeCAD`).
