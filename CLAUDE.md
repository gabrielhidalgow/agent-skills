# agent-skills — working notes

Skills for agentic coding tools. This repo **is** the live installation — see Layout.

Public: https://github.com/gabrielhidalgow/agent-skills

## Layout — there is no build, deploy, or sync step

```
~/Desktop/Projects/agent-skills/pinterest/   ← this repo (edit here)
~/.claude/skills/pinterest  →  symlink to it  ← what /pinterest loads
```

The symlink means both paths are **the same file** (verified by inode). Editing here is live on the very
next `/pinterest` invocation — no reinstall, no copy. `git status` sees edits made through either path.

The only non-automatic step is a **stale clone**: changes made on github.com or another machine need
`git pull` here first. Check with `git status -sb | head -1` → clean is `## main...origin/main`.

## Testing a change

Run `/pinterest` on a real brief. There is no unit test — the skill is instructions, and the only
meaningful check is whether a live run behaves.

**Do not stress-test Pinterest.** It is an unauthenticated scraper against a private API; finding the rate
limit means doing the thing the skill forbids. For layout or formatting changes, rebuild a contact sheet
from thumbnails already in the session scratchpad instead of running a fresh search.

Extract the sheet script from the reference file rather than retyping it, so you test what is actually
written:

```bash
python3 -c "import re,pathlib;print(re.search(r\"<<'PY'\n(.*?)\nPY\n\",pathlib.Path('pinterest/references/contact-sheet.md').read_text(),re.S).group(1))" > /tmp/sheet.py
uv run --quiet --with pillow python /tmp/sheet.py <thumbs-dir> <out.jpg> [start_n]
```

## Invariants — do not break these without a reason

Each was established by measurement or by a live failure, not preference:

- **Search the user's literal words.** A "craft" rewrite was measured to make results *worse*
  (`stadium south america` → real stadiums; `concrete stadium architecture` → parking garages).
- **12 images, one sheet, direction in the same reply.** One round trip is the whole point.
- **Never write into the user's project.** No reference folder, no markdown, no `.gitignore` edit.
- **Sample colour from originals where accuracy matters**, and label thumbnail-sampled values approximate.
- **Say what the set cannot tell you.** With no picking step this is the only guard against over-claiming.

## Gotchas already paid for

- **Every subdirectory of `~/.claude/skills/` registers as a skill — dot-prefixed included.** A backup
  stashed there appears as a duplicate. Keep backups out; git history is the backup.
- **`curl` inside a `while read` loop eats the loop's stdin** and hangs until killed. `</dev/null` is
  load-bearing.
- **Validate downloads by MIME type, never by size.** Roughly one pin in eight has no `/474x/` CDN variant
  and returns an `AccessDenied` **XML body** saved under a `.jpg` name.
- **Sheet aspect is measured, not intuited.** Claude Code's viewer caps near 743×1190, so ~0.6 fills it;
  both "taller" and "more landscape" were tried and measured worse. Re-measure on a different host.
- **A failing run is not automatically an API change.** Worked-then-stopped → likely throttling, wait and
  retry once. Cold-start failure across queries that persists → API changed; patch the pinned fork.

## The engine

`pinterest-dl`, pinned to a commit on a personal fork
(`gabrielhidalgow/pinterest-dl@0dcb2078`, Apache-2.0). Pinned so a fresh machine reproduces this build and
so a Pinterest-side break can be patched without waiting on upstream. A git install reports version
`v0.0.0.dev0` — expected, not broken; identity comes from
`~/.local/share/uv/tools/pinterest-dl/uv-receipt.toml`.

## Committing

Edit → test with a live run → commit → push. The repo is the source of truth; the symlink means there is
nothing else to update.
