# The contact sheets

Numbered grids sent inline in the conversation, between thumbnail fetch and direction. The user's reply
with numbers is the **only** selection mechanism — there is nothing to click, publish, or read back,
which is what makes this flow identical on every surface.

## Building the sheets

**One sheet for the whole set**, laid out as justified rows, **the per-row count derived from how many
images there are** (12 → 3, 24 → 4) so the sheet lands near the viewer's ideal aspect. Each row shares
a single height, widths following each image's true aspect so the row fills the sheet width exactly. Nothing is
letterboxed and nothing is cropped — every image is as large as its row allows. Numbering runs 1–N down
the sheet and must match the `n → pin URL` list in the message.

```bash
uv run --quiet --with pillow python - "$D/thumbs" "$D/sheet.jpg" [start_n] <<'PY'
import sys, pathlib, json
from PIL import Image, ImageDraw, ImageFont

thumbs_dir, out_path = pathlib.Path(sys.argv[1]), pathlib.Path(sys.argv[2])
start_n = int(sys.argv[3]) if len(sys.argv) > 3 else 1   # refine rounds start at 25, 49, ...
order = json.loads((thumbs_dir.parent / "order.json").read_text())[start_n - 1:]
files = [next(thumbs_dir.glob(f"{pid}.*")) for pid in order]

SHEET_W, GAP, MARGIN = 2400, 12, 16
# Per-row derives from the count: pick the one landing nearest the viewer's
# ideal 0.62 aspect. 12 -> 3, 24 -> 4. Hardcoding it wastes half the viewer.
med = sorted(Image.open(f).size[0] / Image.open(f).size[1] for f in files)[len(files) // 2]
PER_ROW = min(range(2, 7), key=lambda c: abs((c * med) / -(-len(files) // c) - 0.62))
BG = (11, 11, 14)
inner = SHEET_W - 2 * MARGIN

# Justified rows: every image in a row shares one height; widths follow true
# aspect so the row fills `inner` exactly. No letterboxing, no cropping.
rows = [files[i:i + PER_ROW] for i in range(0, len(files), PER_ROW)]
sizes = []
for row in rows:
    dims = [Image.open(f).size for f in row]
    ars = [w / h for w, h in dims]
    avail = inner - GAP * (len(row) - 1)
    h = avail / sum(ars)                # rows always fill the full width
    sizes.append([(round(a * h), round(h)) for a in ars])

SHEET_H = 2 * MARGIN + sum(s[0][1] for s in sizes) + GAP * (len(rows) - 1)
sheet = Image.new("RGB", (SHEET_W, SHEET_H), BG)
draw = ImageDraw.Draw(sheet)
try:
    font = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 40)
except OSError:
    font = ImageFont.load_default(size=40)

n, y = start_n, MARGIN
for row, row_sizes in zip(rows, sizes):
    x = MARGIN
    for f, (w, h) in zip(row, row_sizes):
        sheet.paste(Image.open(f).convert("RGB").resize((w, h), Image.LANCZOS), (x, y))
        bx, by, r = x + 12, y + 12, 32          # badge drawn AFTER the image
        draw.ellipse([bx, by, bx + 2 * r, by + 2 * r], fill=(0, 0, 0),
                     outline=(91, 163, 255), width=3)
        tb = draw.textbbox((0, 0), str(n), font=font)
        draw.text((bx + r - (tb[2] - tb[0]) / 2, by + r - (tb[3] - tb[1]) / 2 - tb[1]),
                  str(n), fill=(255, 255, 255), font=font)
        x += w + GAP
        n += 1
    y += row_sizes[0][1] + GAP

sheet.save(out_path, "JPEG", quality=82)
print(out_path, f"{SHEET_W}x{SHEET_H}", f"{out_path.stat().st_size // 1024}KB", f"{len(files)} images")
PY
```

Notes that matter:

- **`order.json` accumulates across refine rounds** — append new ids rather than replacing, and pass
  `start_n` (25 for the first refine, 49 for the next) so numbering continues and never collides. It is the
  single source of numbering — the sheets, the `n → pin URL` list, and the pick-to-id mapping all read
  it, so a number can never point at the wrong pin.
- Thumbnails may be jpg/png/webp after the CDN fallback — `Image.open` handles all three.
- **Rows always fill the full width — never cap the row height at the source size.** Capping produces
  ragged half-empty rows to prevent softness that never appears (see the aspect note below).
- **Shape the sheet to the HOST'S IMAGE VIEWER, not to the window — this is the single biggest lever on
  how large the images look, and intuition has been wrong about it in both directions.** Aim for a sheet
  aspect near **0.6 — mildly portrait**; too tall wastes width, too wide wastes height.

  That 0.6 target is **measured for Claude Code**, whose popover caps around 743×1190. Another host will
  differ. To re-measure on any host: display a sheet of known natural size, screenshot it, and compare
  displayed pixels to natural pixels — the ratio tells you which dimension is the binding constraint.
  Until measured, 0.6 is a sane default. Measured on a real 24-image set in Claude Code:

  | Images | Per row | Sheet aspect | Displayed | Image width |
  |---|---|---|---|---|
  | 24 | 3 | 0.30 | 356×1190 | 116px — wastes width |
  | 24 | **4** | 0.53 | 630×1190 | 153px |
  | 24 | 6 | 1.20 | 743×621 | 119px — wastes height |
  | **12** | **3** | **0.62** | **733×1190** | **239px** |

  Note 3-per-row and 6-per-row are equally bad at 24, for opposite reasons — do not assume "more
  landscape" or "fewer per row" helps without measuring. And halving the count to 12 makes each image
  **56% larger**, because 3-per-row at 12 lands exactly on the ideal aspect.
- **The viewer cannot be resized** — it belongs to the host, not the skill. If the user wants it truly
  large, the sheet is a real file on disk: opening it outside the agent shows it uncapped.
- At 2400px wide and 3 per row each image lands near 790px in-sheet, above the 474px CDN source. That is
  fine: it displays near 239px, so the upscale is never visible.
- The badge outline uses `#5ba3ff` purely for legibility against both dark and light cells.

## The message that carries it

Display the sheet inline by whatever the host provides (Claude Code: `SendUserFile`, `display: render`;
another agent: its equivalent; no image channel at all: print the absolute path). **The direction goes in
the same message** — there is no pick-waiting step; see `SKILL.md` Step 7.

Under the direction, list every source as **`n` — description** linked to its pin. Never bare numbers: a
lone digit is a one-character click target that says nothing about what it points at. Close with the one
line that keeps refinement discoverable:

> Or say **"more like 7"** and I'll search from that pin itself rather than from keywords.

`order.json` is the single source of numbering — the sheet, the source list, and any later refine round
all read it, so a number can never point at the wrong pin. Refine rounds append to it and continue
numbering (13+), never restart at 1.
