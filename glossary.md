# Glossary — Workspace & Gold Image Terms

## Repos

- **workspace-base** — Source repo for the gold image. Holds the Dockerfile that gets built and pushed (eventually to GHCR) as the team's baseline image. This **is** "the gold image."
- **workspace-config** — Shared config repo. Holds the Makefile and `decomk.conf` that decomk reads. Single source of truth for *what* gets installed and in *what order*. Per-project tool sets defined here.
- **fpga-workbench** — A project repo. Example consumer of workspace-base + workspace-config. Holds project code (Verilog, etc.) and a `.devcontainer/` that pins the gold image and runs decomk.
- **decomk** — Steve's tool (stevegt/decomk). Reads `decomk.conf` + Makefile, runs the right targets, writes stamps. *Not ours to edit.*

## Image versions ("blocks")

Naming heritage: ISConf, ~1998. Each block is a Docker image tag, not a repo.

- **block00** — Vendor base + just enough to bootstrap decomk. For us: `mcr.microsoft.com/devcontainers/base:ubuntu` (pinned) + decomk binary. Nothing else.
- **block0** — `block00` + base tools (apt packages, goenv, pyenv, Go, Python). What workspace-base currently builds.
- **block10** — `block0` + the next layer of tools, once block0 is stable and we've grown tired of waiting on installs above it.
- **blockN** — Same pattern. Each block uses the previous block as its FROM line. New block = new image cut, when install time gets long.

**Rule:** never delete an old block tag. Someone's codespace may depend on it.

## Stamps

- **`/var/decomk/stamps/`** — Where decomk/Make writes a touch-file after each install target completes. Make checks for the stamp before re-running. This is what makes the Makefile safe to run twice (once at image-build time, once at container-create time): the second run sees the stamps and is a no-op.

## Config files

- **`decomk.conf`** — Maps project names (e.g. `fpga-workbench`) to tool sets. Front-end to the Makefile. Equivalent role to ISConf's config files.
- **`Makefile`** — Defines every install target, in order, with version pins and stamps. The "what and how" of installation.
- **Dockerfile** (in workspace-base) — Steve's spidey sense: should be ~2 lines. `FROM <previous block>` + `decomk run`. All install logic lives in the Makefile, not here. *Hypothesis, not decided.*

## Roles each repo plays at runtime

1. Developer opens fpga-workbench in a codespace.
2. `devcontainer.json` says `image: workspace-base:blockN` (pinned).
3. Container starts from that image — base tools already present.
4. `postCreateCommand` runs decomk.
5. decomk clones workspace-config, reads `decomk.conf`, runs Makefile targets for `fpga-workbench`.
6. Stamps from the gold image mean shared targets are skipped; only project-specific targets (oss-cad-suite, cocotb, etc.) actually run.

## Not in scope (yet)

- **GHCR** — where block images will eventually be pushed. *Later.*
- **gold-server** — currently the home for this glossary and the gold-image spec doc. Not a runtime component.
- **dev container "features"** — considered and leaned away from (per-feature repo overhead, VS Code dependency).

## Open / unsettled

- **Two-line Dockerfile** — Steve's hypothesis, not decided. Needs to be tried.
- **Variable renames in decomk** — upstream is mid-flux (`tool_mode`, `install_package`/`repo` collapsing). Hold on renames until upstream stabilizes.
- **apt version pinning** — currently unpinned; pin per-package eventually.
