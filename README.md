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

**Any agent — one command.** Works for Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, Gemini CLI,
OpenCode and others:

```bash
npx skills add gabrielhidalgow/agent-skills
```

Add `-g` to install for every project rather than just the current one.

**Claude Code, natively** — register this repo as a plugin marketplace:

```bash
/plugin marketplace add gabrielhidalgow/agent-skills
```

Then `/plugin install pinterest@gabrielhidalgow-agent-skills`.

**By hand** — the skill is a directory of plain markdown, so any agent that reads instructions from a file
can use `skills/pinterest/SKILL.md` directly. The only host-specific line is how to display an image
inline, and it names the fallback (print the file path) for hosts without an image channel.

## Hacking on it

Clone and symlink, so edits are live with no sync step:

```bash
git clone https://github.com/gabrielhidalgow/agent-skills.git ~/code/agent-skills
ln -s ~/code/agent-skills/skills/pinterest ~/.claude/skills/pinterest
```

### Updating

The symlink means the clone and the live skill are **the same file**, not two copies — verified by inode.
So there is no sync command to run locally, and no copy step to forget:

| Where you edited | To update the live skill |
|---|---|
| The clone, or `~/.claude/skills/pinterest` — same file | **Nothing.** It is already live. Commit and push when ready. |
| github.com, or another machine | `git -C ~/code/agent-skills pull` — then live immediately |

The one failure mode is a **silently stale clone**: edit on github.com, forget to pull, and this machine
keeps running the old skill with no error. One-line check:

```bash
git -C ~/code/agent-skills status -sb | head -1     # "## main...origin/main" with no ahead/behind
```

Pulling is deliberately manual rather than hooked, so nothing changes the skill mid-project.

### Installing gotcha

**Every subdirectory of `~/.claude/skills/` registers as a skill — including dot-prefixed ones.** A backup
copy stashed there (`.pinterest-backup`) shows up as a second, duplicate skill. Keep backups outside that
directory; the git history is the backup anyway.

### Not usable in Claude chat

Agent Skills there run in a sandbox with **no network access and no runtime package installation**, so the
scraper can neither install nor reach Pinterest. Fetching Pinterest search with the native web tools does
not work either — the results are JS-rendered behind a login wall and come back as an empty shell.

### Scope

`pinterest-dl` is an unofficial scraper; automated access can conflict with Pinterest's ToS. The skill
stays at research scale — a query or two per task, 12 images, no bulk harvesting, no scheduled runs, and
public pins only. References inform direction; they are third-party copyrighted work and never ship inside
a deliverable.
