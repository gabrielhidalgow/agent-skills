# Query craft + what to report back

## Read this first: which kind of search is this?

**The user's words run first, verbatim, always.** This file is for the case where those words genuinely
underperform and you need to *propose* an alternative — never for pre-emptively "improving" what they typed.

Everything below applies to ONE of two kinds of search, and applying the wrong set makes results worse:

| | **Subject search** | **Treatment search** |
|---|---|---|
| You want | Images **of** a thing — stadiums, terraces, a city, a material | To see how designers **handle** something — posters, covers, packaging |
| Query shape | The plain words for the thing. Nothing else | Artifact noun + style/medium |
| Example | `stadium south america` | `swiss typographic poster` |
| The rules below | **Do not apply.** Extra nouns dilute the subject | Apply fully |

**Measured evidence, not theory.** For a subject search, adding craft words actively destroyed relevance:

| Query | Usable | What came back |
|---|---|---|
| `stadium south america` | 23 | Rio, Santiago de Chile, aerial bowls, packed terraces |
| `concrete stadium architecture` | 21 | Parking garages, escalators — 3 of 12 were stadiums, none South American |

Same volume, opposite relevance. `concrete` and `architecture` are enormous generic buckets that drown out
`stadium`. When someone wants pictures of a thing, name the thing and stop.

---

# Treatment searches: the craft rules

Everything from here down applies **only** when the goal is to see how something is designed.

## How Pinterest search actually behaves


Pinterest matches against pin titles, descriptions, board names, and alt text — not against image
content. So the query has to sound like how a *person who saved that pin* would have labelled it.

**Works:**
- Concrete medium + style: `swiss typographic poster`, `risograph zine spread`, `brutalist album cover`
- Named movement or era: `bauhaus exhibition poster`, `90s rave flyer`, `art deco menu`
- Material or process: `letterpress business card`, `duotone print`, `embossed packaging`
- Composition language: `editorial grid layout magazine`, `asymmetric poster layout`
- Colour **anchored to an artifact**: `risograph print colour palette`, `book cover warm neutral`

**Returns generic slop:**
- Adjective soup with no medium: `beautiful modern clean design`
- Brief language nobody pins with: `visual identity for a premium consulting firm`
- Over-long queries — past ~5 words, matching degrades fast
- Your client's or project's name

**The colour trap — verified, not theoretical** (a treatment-search failure). A query built from mood plus colour with no artifact
noun drifts straight into abstract stock art. `muted warm neutral palette print` returned eight
abstract paintings and paint-swatch stock photos — zero graphic design. The word "palette" is pinned
overwhelmingly by painters and interior accounts, and "print" reads as wall art, not printed matter.
Always anchor colour queries to the thing you are designing: `book cover warm neutral`,
`packaging muted earth tones`, `risograph poster two colour`. If a query comes back as paintings,
photographs of paint, or interiors, that is the signal — rewrite it with an artifact noun rather than
salvaging the results.

**Photographic drift, same family of failure** (also treatment-only — for a subject search, photographs of the subject are exactly what you want). Queries about light, texture or material
(`caustics`, `refraction`, `long exposure`) pull photographs of the phenomenon rather than design that
uses it. Photographs of water, glass and people rarely survive as a background behind type, and they
carry licensing the project cannot use. Keep the artifact noun in the query, and reject on sight.

## Building a set of two or three treatment queries

Cover different axes so the set does not collapse into one look:

| Axis | Example for "editorial site for a design studio" |
|---|---|
| Layout / composition | `editorial grid layout magazine spread` |
| Typography | `swiss typographic poster` |
| Colour / texture | `book cover warm neutral` |
| Art direction / mood | `minimal studio photography editorial` |

Two strong queries beat four vague ones — and with a ten-image budget, four queries leaves barely two
keepers each. Prefer two or three, and spend the budget where the brief is least settled.

## Working from a board or pin instead of a query

When the user gives a Pinterest URL, use `scrape` rather than `search` — with no `-o`, it screens for
free exactly like a search:

```bash
pinterest-dl scrape "<board_or_pin_url>" -n 25 --json
```

For a pin URL this returns that pin plus related pins — a good way to widen from one reference the user
already likes. `--related-only` skips the original and returns only the recommendations. Public boards
and pins only.

## Reading images for direction

For each image, extract what is **transferable**, not what is decorative:

- **Palette** — pick actual hex values. Note the ratio: what dominates, what accents, at roughly what
  proportion. A palette without proportions is just a swatch list, and proportion is usually where the
  real finding hides — a reference set can share a hue with the current design and still differ
  completely in how much of the surface each value occupies.
- **Type** — classification (grotesk, transitional serif, mono, display), weight contrast, case,
  tracking, how many sizes are in play. Describe the treatment; name a family only when genuinely
  identifiable, never as a guess presented as fact.
- **Layout** — the grid underneath, margin generosity, alignment logic, where the eye enters and
  where it goes second.
- **Texture** — grain, paper, print artefacts, gradients, noise, and whether they are structural or
  surface polish.
- **The tension** — most strong references hold one deliberate tension: huge against tiny, dense
  against empty, warm against cold. Name it. That tension is usually the thing worth borrowing.

## What to report back

The reply *is* the deliverable. Shape it for the task at hand — a poster brief does not need motion
notes, a screen brief does not need ink coverage. Lead with the finding, not the process.

1. **The finding, in one or two sentences.** The single most useful thing the references revealed about
   the work in progress. If measurement supports it (sampled hex, proportions), give the numbers —
   a measured claim survives argument in a way an impression does not.
2. **The guidelines that follow from it** — palette, composition, type, texture, motion as the brief
   requires. Specific enough to act on.
3. **What to avoid** — how this direction fails when executed badly.
4. **The references behind it** — the pins you actually drew on, each with its URL, and one line on
   what it contributed. Not a gallery; only the ones carrying an argument.
5. **The connection to the work** — which files, tokens, or components each point touches, and an offer
   to apply it.

Keep it honest: if the set was weak, say the set was weak. A confident direction built on eight
rejected pins is worse than saying the queries need rewording.
