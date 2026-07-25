# Custom Art Guide

Every new resource/room in this mod so far deliberately reuses vanilla art
(Cotton/Fabric/Clothes sprites, Weaver furniture) so mechanics could be
tested without blocking on art. This doc covers what real art you'd need
per asset, and the safest way to make it. See `doc/textiles/07_implementation_log.md`
for the current list of placeholder-art resources/rooms still needing real
art.

## The three asset kinds a new raw resource needs

Every resource referenced by a `SPRITE:` or `ICON:` key in
`assets/init/resource/*.txt` needs matching files under
`assets/sprite/...`. For a growable fiber (Cotton's own pattern), that's:

| Purpose | Path | Real vanilla example | Actual pixel size |
|---|---|---|---|
| Stockpile/pile sprite | `assets/sprite/resource/<Name>.png` | `Cotton.png` | 244 × 94 |
| Field/growing-plant sprite | `assets/sprite/resource/growable/<Name>.png` | `Cotton.png` | 460 × 34 |
| Small UI icon | `assets/sprite/icon/24/resource/<Name>.png` | `Cotton.png` | 72 × 36 |

Note the icon file is **not** a single 24×24 square — it's a horizontal strip
(72 wide ÷ 24 = 3 frames, one per fill-level or similar; the `ICON:
24->resource->Cotton->0` key's trailing `->0` picks frame 0 from that strip).
The pile and growable sprites are also multi-frame strips (multiple growth
stages / pile amounts laid out left-to-right), and the exact frame-slicing
convention isn't spelled out in `doc/config` — it's easiest to reverse-engineer
by opening the vanilla PNG in an image editor and eyeballing where the frame
boundaries fall, since frame width is implied by usage rather than documented.

**The safe way to make new art:** duplicate the vanilla PNG you're replacing,
keep its exact canvas dimensions and frame grid, and only redraw the pixel
content of each frame. This guarantees the `SPRITE:`/`ICON:` frame-index
references in your config still line up, without needing to reverse-engineer
the slicing math yourself.

## What a refining room's art actually involves

This mod's `REFINER_*`/`WORKSHOP_*` rooms all copy their `SPRITES:` block
verbatim from the closest vanilla room, referencing frame indices from
shared spritesheets under `assets/sprite/game/combo/` (e.g. `REFINER.png`,
288 × 576 — shared by *all* refiner rooms, indexed by frame number). This is
substantially more art than a resource icon: main machine idle/working
frames, top overlay frames, conveyor/storage furniture, potentially
animated (`FPS`, `CIRCULAR`) and multi-directional (`ROTATES`).

Options, cheapest to most expensive:

1. **Keep reusing vanilla furniture art** (current state for every room) —
   zero art work, building looks like a generic loom/refinery.
2. **Recolor existing frames** — duplicate the sheet, tint it a different
   hue per material. Cheap, gives each building a distinct identity without
   new poses.
3. **Full custom spritesheet** — new frames matching the same grid layout,
   registered as a new sheet path. Real pixel-art production work; do this
   only once mechanics are fully proven.

## Order of operations recommendation

Finish proving out all chains (Plant Fibre, Wool, Silk, Dye) mechanically
with placeholder/recolored art first, then do one dedicated art pass across
all of them together - avoids re-learning spritesheet conventions per
material and avoids wasted art if a mechanical redesign changes which rooms
exist.
