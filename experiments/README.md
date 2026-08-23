# experiments

Scratch space for trying things on the skill. **Git-ignored** — nothing here is committed or pushed.

## Why it sits here and not inside `pinterest/`

- **Outside the skill folder**, so it can never affect how `/pinterest` loads. Sub-directories of a skill
  (like `pinterest/references/`) are read as part of that skill; anything you leave in there travels with
  it. Here, nothing does.
- **Inside the repo directory**, so it is right where you already work — no second folder to open.
- **Ignored by git**, so half-finished experiments never reach the public repo. `.gitignore` has
  `experiments/`.

Note the asymmetry that caused a real bug once: **top-level** directories under `~/.claude/skills/`
each register as a skill, dot-prefixed ones included — a backup copy left there showed up as a duplicate.
Sub-directories do not. That is why this lives at the repo root rather than anywhere under the symlink.

## Suggested shape

One directory per experiment, dated, with a note on what you were testing and what happened:

```
experiments/
  2026-08-24-sheet-density/
    NOTES.md          what was tried, what the result was
    …                 images, scripts, whatever
```

The notes matter more than the artefacts. Most of what this skill knows — literal queries beating
rewrites, 0.6 sheet aspect, MIME-not-size validation — came from a measurement someone could have
forgotten. If an experiment settles something, promote the conclusion into `pinterest/SKILL.md` or
`CLAUDE.md`; the experiment folder itself stays disposable.

## Promoting something out

If an experiment turns into a keeper, `git add -f <path>` overrides the ignore — but prefer moving the
useful part into the skill or the working notes rather than committing scratch.
