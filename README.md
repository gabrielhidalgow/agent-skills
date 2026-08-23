# agent-skills

Skills for agentic coding tools. Plain markdown instructions — nothing here is tied to one vendor.

## `pinterest` — quick visual reference for design decisions

Searches Pinterest with your own keywords, shows 12 numbered thumbnails as a single contact sheet, and
delivers the design direction — palette, composition, type, texture — **in the same reply**, so you can
carry straight on with the work. One round trip. Writes nothing into your project.

### Requirements

Any agent with a shell: **bash**, `curl`, `jq`, `file`, and **`uv`** (or any Python with Pillow).

The engine is a pinned fork of [sean1832/pinterest-dl](https://github.com/sean1832/pinterest-dl)
(Apache-2.0). The skill installs it on first run:

```bash
uv tool install "pinterest-dl[image] @ git+https://github.com/gabrielhidalgow/pinterest-dl@0dcb2078f968460230d23b25e8179a1eb9f29e11"
```

Pinned to a release commit on a personal fork so a fresh machine reproduces the same build, and so a
Pinterest-side breakage can be patched immediately without waiting on upstream.

### Install

**Claude Code** — symlink into the user-level skills directory so it loads in every project:

```bash
git clone https://github.com/gabrielhidalgow/agent-skills.git ~/code/agent-skills
ln -s ~/code/agent-skills/pinterest ~/.claude/skills/pinterest
```

**Any other agent (Codex, Grok CLI, …)** — point it at `pinterest/SKILL.md` as instructions. The file is
plain markdown; the only host-specific line is how to display an image inline, and it names the fallback
(print the file path) for hosts without an image channel.

### Not usable in Claude chat

Agent Skills there run in a sandbox with **no network access and no runtime package installation**, so the
scraper can neither install nor reach Pinterest. Fetching Pinterest search with the native web tools does
not work either — the results are JS-rendered behind a login wall and come back as an empty shell.

### Scope

`pinterest-dl` is an unofficial scraper; automated access can conflict with Pinterest's ToS. The skill
stays at research scale — a query or two per task, 12 images, no bulk harvesting, no scheduled runs, and
public pins only. References inform direction; they are third-party copyrighted work and never ship inside
a deliverable.
