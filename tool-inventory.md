# Gold Image Tool Inventory
*Status: Draft — pending team confirmation*
*Last updated: 2026-04-12*
*Base image: Ubuntu 24.04 LTS (Noble Numbat), pinned by sha256*
*Base digest: `mcr.microsoft.com/devcontainers/base:ubuntu@sha256:4bcb1b466771b1ba1ea110e2a27daea2f6093f9527fb75ee59703ec89b5561cb`*

---

## How to use this document

Each tool listed below shows:
- The confirmed or candidate version
- Which layer it belongs to (gold image or decomk)
- Which repo and file it goes in
- Where in that file to add it

Items marked **CONFIRM WITH TEAM** need a version decision before implementation.
Items marked **CONFIRM WITH CONTAINER** need versions pulled from the running image.

To get exact apt versions from the container, run:

```sh
docker run --rm ghcr.io/ciwg/workspace-base:2026-04-07 \
  apt-cache policy git wget jq make build-essential 2>/dev/null | \
  grep -A1 "Installed:"
```

---

## Gold Image (Dockerfile)

These tools are baked into the base Docker image. Installed for every
developer on every machine, regardless of project.

**Repo:** `computerscienceiscool/workspace-base` (transfer to `ciwg/workspace-base` when ready)
**File:** `Dockerfile`
**Where in file:** In the `apt-get install` block, after existing system packages

---

### Already in the Dockerfile — versions confirmed from build 2026-04-07

| Tool | Pinned version | Install method | Notes |
|------|---------------|----------------|-------|
| vim | `2:9.1.0016-1ubuntu7.10` | apt | Confirmed from Docker build |
| neovim | `0.9.5-6ubuntu2` | apt | Confirmed from Docker build |
| openssh-client | `1:9.6p1-3ubuntu13.15` | apt | Confirmed from Docker build |
| curl | `8.5.0-2ubuntu10.8` | apt | Confirmed from Docker build |
| python3-pip | `24.0+dfsg-1ubuntu1.3` | apt | Confirmed from Docker build |
| Go | `1.24.13` | goenv | Pinned explicitly |
| Python | `3.12.13` | pyenv | Pinned explicitly |

### All apt pins confirmed from noble archive (2026-04-12)

All CONFIRM WITH CONTAINER items have been resolved and pinned in
`workspace-config/Makefile` TOOLS target.

| Tool | Pinned version | Install method | Notes |
|------|---------------|----------------|-------|
| wget | `1.21.4-1ubuntu4.1` | apt | Pinned in Makefile |
| git | `1:2.43.0-1ubuntu7.3` | apt | Pinned in both Dockerfile (build-time) and Makefile |
| jq | `1.7.1-3ubuntu0.24.04.1` | apt | Pinned in Makefile |
| make | `4.3-4.1build2` | apt | Pinned in both Dockerfile and Makefile |
| build-essential | `12.10ubuntu1` | apt | Meta-package; pinned as-is |
| libssl-dev | `3.0.13-0ubuntu3.9` | apt | pyenv build dep |
| zlib1g-dev | `1:1.3.dfsg-3.1ubuntu2.1` | apt | pyenv build dep |
| libbz2-dev | `1.0.8-5.1build0.1` | apt | pyenv build dep |
| libreadline-dev | `8.2-4build1` | apt | pyenv build dep |
| libsqlite3-dev | `3.45.1-1ubuntu2.5` | apt | pyenv build dep |
| libffi-dev | `3.4.6-1build1` | apt | pyenv build dep |
| liblzma-dev | `5.6.1+really5.4.5-1ubuntu0.2` | apt | pyenv build dep |
| golang-go | `2:1.22~2build1` | apt | Build-time only — bootstraps decomk; the runtime Go is via goenv |
| ca-certificates | `20240203` | apt | Build-time only |

---

### To be added to the Dockerfile

**Where in file:** Add to the `apt-get install` block. Each tool may need its
own PPA added before the install block.

#### Office

| Tool | Version | Install method | Status |
|------|---------|----------------|--------|
| LibreOffice | `26.2.2` | apt (official PPA or distro repo) | Ready to add |

#### Graphics / Design

| Tool | Version | Install method | Status |
|------|---------|----------------|--------|
| Inkscape | `1.4.3` | apt (official stable PPA) | Ready to add |
| OpenSCAD | `2026.02.10` | apt (OBS repo, pinnable) | Ready to add |

#### EDA / Electronics Design

| Tool | Version | Install method | Status |
|------|---------|----------------|--------|
| KiCad | `9.0.8` **OR** `10.0.0` | apt (version-specific PPA) | **CONFIRM WITH TEAM** |

KiCad 10.0.0 was released March 2026. KiCad 9.0.8 is the previous stable.
Ask your FPGA developer which they prefer. Both are pinnable via PPA.

#### Browser

| Tool | Version | Install method | Status |
|------|---------|----------------|--------|
| Chromium | CONFIRM WITH TEAM | apt (distro repo, pinnable) | Confirm exact version with team |

---

## decomk-managed (Makefile + decomk.conf)

These tools cannot be pinned via apt. Installed by decomk using their own
install mechanisms. Each needs a Makefile target with a stamp file.

**Repo:** `ciwg/workspace-config`
**File:** `Makefile` — add new targets after existing TOOLS/GO/PYTHON targets
**File:** `decomk.conf` — add to DEFAULT or project-specific context as appropriate

---

| Tool | Version | Install method | Notes |
|------|---------|----------------|-------|
| Fritzing | `1.0.6` | AppImage from fritzing.org | No maintained PPA. Released Oct 21, 2025. |
| Ink/Stitch | `v3.2.2` | .deb from GitHub releases | Inkscape must be installed first. Installs as Inkscape extension. |

---

## Versions still needed from team

Collect these at the Thursday team meeting.

| Tool | Question |
|------|----------|
| Go | Which versions beyond 1.24.13? (1.18, 1.22, 1.25 mentioned — confirm all needed) |
| Python | Which versions beyond 3.12? How many? |
| Node | Which version(s)? |
| Rust | Which version(s)? |
| Chromium | Confirm exact version to pin |
| KiCad | 9.0.8 or 10.0.0? |
| neovim | Confirm 0.9.5 is acceptable or upgrade needed |

---

## Open decisions (for team lead)

These require a decision before the next gold image build.

1. Long-term registry location — an image already exists at `ghcr.io/ciwg/workspace-base:2026-04-07` (pre-dates the pinning work). Steve said GHCR is "LATER" — clarify with him.
2. ~~apt package version pinning~~ — **DONE** (2026-04-12). All apt packages now pinned.
3. Full tool inventory — is this list complete? Any tools missing?
4. decomk version pin — `@latest` in `Dockerfile` and `postCreateCommand.sh` needs a specific tag once stevegt cuts a stable release.

---

## Session artifacts (2026-04-12)

Other files in this repo created or updated this session:

| File | Purpose |
|------|---------|
| `status.md` | High-level progress log against `server-todo.md` |
| `server-todo-progress.md` | Inline-annotated copy of the original 1:1 todo with done / partial / not-done marks |
| `why-order-matters-slides.md` | Thursday presentation (remark.js markdown) |
| `transcript_041026.txt` | Raw 1:1 transcript from 2026-04-10 |
| `gold-server/gold-image-spec.md` | Canonical gold-image spec (was duplicated in 3 places) |
| `gold-server/glossary.md` | Names cheat sheet: workspace-base vs workspace-config vs block00/block0/block10 vs decomk |
| `gold-server/features-research.md` | Analysis of dev container features as alternative/complement to decomk |

## References

- Gold image spec: `computerscienceiscool/gold-server` → `gold-image-spec.md`
- Glossary (naming cheat sheet): `computerscienceiscool/gold-server` → `glossary.md`
- Dockerfile (gold image build): `computerscienceiscool/workspace-base` → `Dockerfile`
- decomk config: `computerscienceiscool/workspace-config` → `decomk.conf` and `Makefile`
- Project consumer example: `ciwg/fpga-workbench` branch `decomk-setup` → `.devcontainer/`
- decomk source (not ours): `stevegt/decomk` — local clone at `~/lab/decomk`
- infrastructures.org gold server model: http://www.infrastructures.org/bootstrap/gold.shtml
- Why Order Matters (LISA 2002): https://www.usenix.org/legacy/publications/library/proceedings/lisa98/traugott.html
