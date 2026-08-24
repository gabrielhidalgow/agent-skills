---
name: pinterest
description: Get quick visual reference from Pinterest to ground design decisions in the work in progress. Searches with the user's own keywords, screens candidates for free, shows 12 numbered thumbnails as one contact sheet in the conversation, and delivers the design direction — palette, composition, type, texture — in the same reply, so the project can move straight on. One round trip. Writes nothing into the project, ever. Use when asked to search Pinterest, find visual references, gather inspiration, get design ideas, explore a visual or graphic direction, research a style, or pull references from a Pinterest board URL. For UI/UX patterns and product screens use refero-design or the Mobbin MCP instead — this is for graphic and art direction.
license: MIT
compatibility: Requires a shell with bash, curl, jq, file and uv (or any Python with Pillow), plus network access to install the scraper and reach Pinterest. Not usable in the Claude chat sandbox, which has neither network access nor runtime package installation.
---

# Pinterest visual references

**Purpose: give yourself visual grounding for a design decision, fast.** The user is mid-project and needs
reference to inform what gets built next — not a moodboard to curate, not a document to file.

One round trip: search their words → 12 thumbnails on one sheet → the direction, in the same reply.

## Boundaries — read once, apply always

- **The user's words are the query.** Never silently rewrite them. See Step 3 — this is the rule most
  easily broken with good intentions, and breaking it produces worse results.
- **12 thumbnails per run.** Not a limitation: at 12 the sheet hits the viewer's ideal aspect and each
  image renders **56% larger** than it did at 24. Fewer references, more legible.
- **Nothing is ever written into the project.** Everything lives in the session scratchpad. No reference
  folder, no markdown file, no `.gitignore` entry. The deliverable is your reply.
- **Deliver direction in the same message as the sheet.** Do not stop to ask which images they like —
  that costs a round trip, and the point is speed. They can redirect afterwards if you read it wrong.
- **References are direction, not assets.** Every pin is third-party copyrighted work. It informs palette,
  composition, and treatment. It never ships inside a deliverable, is never traced, and is never fed to an
  image generator as a style target. Abstract the principle; never say "copy this artist."
- **Cite what you looked at.** Every image gets its pin URL in your reply.
- **Stay at research scale.** `pinterest-dl` is an unofficial scraper, and automated access can conflict
  with Pinterest's ToS. A query or two per task. No bulk harvesting, no scheduled or looping runs.
- **Public only.** Never run `pinterest-dl login`. Never create or read a `cookies.json`.

## Requirements

Any agent with a shell can run this — Claude Code, Codex, a Grok CLI, anything with the same capabilities.
The host must provide: **bash**, `curl`, `jq`, `file`, and **`uv`** (or any Python with Pillow). Everything
else the skill installs or fetches itself.

## Step 1 — Preflight

The engine is Gabriel's own fork, pinned to a specific commit — never an unpinned PyPI install:

```bash
command -v pinterest-dl || uv tool install \
  "pinterest-dl[image] @ git+https://github.com/gabrielhidalgow/pinterest-dl@0dcb2078f968460230d23b25e8179a1eb9f29e11"
```

**Check the source, not the version.** `pinterest-dl --version` prints `v0.0.0.dev0` from a git install —
upstream stamps the real version during its release pipeline, so this is expected, not broken. The
authoritative identity is the pin:

```bash
grep requirements ~/.local/share/uv/tools/pinterest-dl/uv-receipt.toml
```

That must name `gabrielhidalgow/pinterest-dl` and the commit above.

**When Pinterest breaks it.** A run returning `{"error": ...}` or empty results usually means Pinterest
changed its private API. That is what the fork is for: check
[upstream](https://github.com/sean1832/pinterest-dl) for a fix, merge it, move the pin — or patch the fork
directly. Never work around it by switching back to the unpinned PyPI package.

*Engine: a fork of [sean1832/pinterest-dl](https://github.com/sean1832/pinterest-dl), Apache-2.0.*

## Step 2 — Anchor to the work in progress

Say in one line what is being designed and what the reference is for. The direction you deliver is
direction **for that** — a hero, a logo, a poster series, a section of a page.

If there is genuinely no work in progress, ask what it is for. Without it you produce pretty pictures and
no decisions.

## Step 3 — Search the user's words, literally

**The query is the user's own phrase, verbatim.** Do not add medium nouns, do not "improve" it, do not
substitute your own vocabulary. If they gave several phrases, run them as given.

This rule exists because breaking it was measured to make results *worse*:

| Query | Usable | What came back |
|---|---|---|
| `stadium south america` (the user's words) | 23 | Rio, Santiago de Chile, aerial bowls, packed terraces |
| `concrete stadium architecture` (a "craft" rewrite) | 21 | Parking garages, escalators — **3 of 12** were stadiums, none South American |

Generic words like `concrete` and `architecture` are enormous buckets that drown the subject; plain phrases
match what people actually tag. If the literal query genuinely underperforms, *propose* a variant and show
the comparison — never swap it on your own initiative. `references/query-craft.md` covers when rewriting
helps and when it hurts.

## Step 4 — Screen for free

`pinterest-dl search` with **no `-o`** downloads nothing — it just prints metadata.

```bash
D="<scratchpad>/pinterest"; mkdir -p "$D"
pinterest-dl search "<the user's phrase>" -n 40 --json > "$D/sweep.json"
jq -r '.results[0].items[] | select(.resolution.x >= 700) | "\(.id)\t\(.resolution.x)x\(.resolution.y)\t\(.alt // "-")"' "$D/sweep.json"
```

Each item carries `id`, `src`, `alt`, `origin` (the pin URL) and `resolution`.

**Alt text is a weak filter** — auto-generated and often wrong about what the image is ("an old book with
some type of font on it" for a Bayer Universal specimen). It screens out obvious misses; nothing finer.

If a sweep returns `{"error": ...}` or an empty array, Pinterest changed its private API. Say so plainly
and stop — never reconstruct a direction from alt text alone.

## Step 5 — Fetch 12 thumbnails

Pinterest's CDN serves size variants by path substitution — `/originals/` → `/474x/` returns about **85 KB
instead of 456 KB**.

```bash
mkdir -p "$D/thumbs"
jq -r '.results[0].items[] | select(.resolution.x >= 700) | [.id, .src] | @tsv' "$D"/sweep*.json \
  | head -12 > "$D/picks.tsv"
jq -R -s 'split("\n") | map(select(length>0) | split("\t")[0])' "$D/picks.tsv" > "$D/order.json"
while IFS=$'\t' read -r id src; do
  curl -sL --max-time 20 -A "Mozilla/5.0" -o "$D/thumbs/$id.jpg" "$(echo "$src" | sed 's|/originals/|/474x/|')" </dev/null
done < "$D/picks.tsv"
```

The `</dev/null` is **load-bearing**: inside a `while read` loop curl otherwise eats the loop's stdin and
hangs until killed.

**Validate by file type, never by size — this bites on roughly one pin in eight.** Some pins have no
`/474x/` variant and the CDN answers with an `AccessDenied` **XML body**, so curl saves a non-empty file
with a `.jpg` name that is not an image:

```bash
for f in "$D/thumbs/"*; do
  [ "$(file -b --mime-type "$f")" = "image/jpeg" ] && continue
  id=$(basename "${f%.*}")
  src=$(jq -r --arg id "$id" '.results[0].items[] | select(.id == $id) | .src' "$D"/sweep*.json | head -1)
  curl -sL --max-time 30 -A "Mozilla/5.0" -o "$f" "$src" </dev/null
done
```

## Step 6 — Look at all 12

Read the sheet (Step 7 builds it) or the thumbnails directly. You are extracting the direction yourself —
there is no picking step — so look properly. While you look, write a **short factual label per image**
("Bayer Universal alphabet specimen", "concentric-circle exhibition poster"). These become the source
links. They name the subject from a contact-sheet view, so they describe subject and composition, not
fine detail.

## Step 7 — Sheet and direction, in one reply

Build **one contact sheet** with the Pillow script in `references/contact-sheet.md` (12 images → 3 per
row) and **display it inline by whatever mechanism the host provides** — in Claude Code that is
`SendUserFile` with `display: render`; in another agent, its equivalent image-display call. If the host
cannot display images at all, print the sheet's absolute path so the user opens it themselves; never
proceed as though they have seen it.

**In the same message**, give the direction. Keep it short enough to act on immediately:

- **Palette** — sampled hex with the role and rough proportion of each. Sample from the thumbnails; there
  is no full-resolution fetch. **Label the values approximate** — a 474px re-encode barely moves dominant
  colours on flat graphic work but drifts on photographic gradients, and quoting an approximation as exact
  is the failure to avoid.
- **Composition** — grid logic, margins, scale relationships, where the eye enters.
- **Typography** — classification, weight and contrast behaviour, case, spacing. Describe the treatment;
  name a family only when genuinely identifiable, never as a guess presented as fact.
- **Texture** — treatment, grain, print artefacts, and whether they are structural or surface polish.
- **What to avoid** — how this direction fails when executed badly.
- **What the set cannot tell you** — with no picking step this is the only guard against over-claiming.
  If the 12 carry no type reference, say so instead of inventing one.

Then the sources, as **`n` — description** links, never bare numbers (a lone digit is a one-character
click target that says nothing), and one line offering refinement:

> Or say **"more like 7"** and I'll search from that pin itself rather than from keywords.

Then connect the direction to the actual files, tokens or components in play, and carry on with the work
if that is what was asked. Do not wait for approval of the direction itself.

## Step 7b — Refine from a pick ("more like 7")

When the user wants more of one image, widen using **Pinterest's own related-pins graph** seeded by that
pin — never keywords you derive from the image yourself:

```bash
pinterest-dl scrape "<pin url>" -n 40 --related-only --json > "$D/sweep-related.json"
```

Dedup by id against everything already shown, **continue numbering from the high-water mark** (a second
sheet starts at 13, not 1 — reused numbers make `n → pin` ambiguous), append to `order.json`, then deliver
sheet + direction exactly as Step 7.

*"More like 7"* → related-pins. *"Like 7 but in blue"* → a **new literal query** (Step 3), because it
introduces a word the related-graph cannot know.
