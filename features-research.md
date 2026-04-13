# Dev Container Features — Research Notes

Written for the "why not just use dev container features?" question
raised in the 2026-04-10 1:1. Answers whether features could replace or
complement decomk + gold-image.

## What dev container features are

A **feature** is a small, shareable unit of devcontainer setup defined
by the dev containers spec (containers.dev). It's a directory that
contains:

- `devcontainer-feature.json` — metadata + options
- `install.sh` — script that runs inside the container at build time

Features are referenced from `devcontainer.json`:

```json
"features": {
  "ghcr.io/devcontainers/features/go:1": { "version": "1.24.13" },
  "ghcr.io/devcontainers/features/python:1": { "version": "3.12" }
}
```

When the container is built (or postCreate runs, depending on the
runtime), the dev containers CLI downloads each feature's OCI artifact,
runs its `install.sh`, and the result is baked into the container.

Features are published as **OCI artifacts in a registry** (GHCR,
Docker Hub, etc.). Public registries exist at
`ghcr.io/devcontainers/features/...` for common tools.

## How they would map to our problem

We currently install via workspace-config/Makefile (decomk driven).
Features would be an alternate path:

| Our target | Public feature (hypothetical) |
|------------|--------------------------------|
| TOOLS (apt) | `ghcr.io/devcontainers/features/common-utils` (subset) |
| GO (goenv + Go 1.24.13) | `ghcr.io/devcontainers/features/go` |
| PYTHON (pyenv + Python 3.12) | `ghcr.io/devcontainers/features/python` |
| OSS (oss-cad-suite 2026-03-07) | *no public feature — we'd write one* |
| I2C (reference clone) | *no public feature — we'd write one* |
| COCOTB | *no public feature — we'd write one* |

## What Steve flagged against

Two concerns from the 1:1:

1. **Per-feature repo overhead** — each custom feature (OSS, I2C,
   COCOTB) would need to be published as an OCI artifact, which in
   practice means a separate repo or a mono-repo with a feature-per-dir
   convention, plus a publish workflow to push OCI artifacts on every
   change. That's repo organization cost the team would have to carry.

2. **VS Code / dev containers CLI dependency** — features are resolved
   and applied by the dev containers CLI (used by VS Code, GitHub
   Codespaces, devpod, etc.). Running the container outside one of
   those environments (plain `docker run`, other IDEs, CI) means the
   features never execute. The base image is then missing everything.
   Our decomk-driven setup runs identically regardless of how the
   container is invoked.

## What features do well

- Low friction to consume public, common tools (Node, Go, Python,
  Docker-in-Docker, kubectl, etc.).
- Options (versions, flags) declared in JSON instead of shell.
- Well-integrated with Codespaces prebuild caching.

## What features do poorly for our case

- **Reproducibility**: features often pull "latest" for their tool
  unless a version option is given, and even pinned versions depend on
  the feature's own install script which can change over time.
- **Ordering**: features declare install order loosely
  (`installsAfter`); no equivalent to Make's explicit prereq graph.
- **Stamp/idempotency**: features re-run fully each build. They don't
  have decomk's stamp dir pattern that lets the same Makefile run at
  image-build time AND at container-create time as a no-op.
- **Policy-per-project**: we map project repos to tool sets via
  `decomk.conf`. Features don't have a corresponding abstraction; each
  project repo's devcontainer.json would list its own features with
  its own options, duplicated across repos.
- **Block-numbered image cuts**: features don't participate in the
  infrastructures.org progressive-image model. They install fresh
  every time (or rely on Codespaces prebuild caching, which is a
  separate system).

## Verdict

**Don't replace decomk with features.** The reproducibility,
ordering, and stamp-idempotency story is better with decomk + Make.

**Don't use features to install our shared tools** (Go, Python, apt
packages). Those are in the gold image, with pinned versions Steve can
audit in one place. Adding features for the same work creates a second
source of truth.

**Do consider features for tightly-scoped, per-project optional tools**
that don't merit a Makefile target and aren't shared across projects.
Example: if one repo needed a specific Docker-in-Docker configuration
and no other repo did, using the public `docker-in-docker` feature in
just that project's devcontainer.json is cheaper than adding a Make
target. This case hasn't come up yet for us.

## If we ever did publish our own features

Rough shape of a minimal feature repo:

```
our-features/
  src/
    oss-cad-suite/
      devcontainer-feature.json
      install.sh
  .github/workflows/
    release.yml    # publishes OCI artifacts on tag
```

The publish workflow uses `devcontainers/action@v1` and pushes to
`ghcr.io/ciwg/features/<name>:<version>`. This would require GHCR
access (same as the gold image), which is out of scope for now.

## References

- https://containers.dev/implementors/features/
- https://github.com/devcontainers/features (public reference features)
- https://github.com/devcontainers/feature-starter (template)
