# Custom Art Guide

Flax v1 deliberately reuses vanilla art (Cotton's sprites for FLAX, Fabric's
sprites for LINEN, Weaver's furniture for the Retting Shed) so the mechanics
could be tested without blocking on art. This doc covers what real art you'd
need per asset, and the safest way to make it.

## The three asset kinds a new raw resource needs

Every resource referenced by a `SPRITE:` or `ICON:` key in
`assets/init/resource/*.txt` needs matching files under
`assets/sprite/...`. For a growable fiber (flax, hemp, and cotton's own
pattern), that's:

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

`REFINER_RETTER_FLAX.txt`'s `SPRITES:` block (copied verbatim from
`REFINER_WEAVER.txt`) references frame indices from several shared spritesheets
under `assets/sprite/game/combo/` (e.g. `REFINER.png`, 288 × 576 — a sheet
shared by *all* refiner rooms, indexed by frame number per room). This is
substantially more art than a resource icon: main machine idle/working frames,
top overlay frames, conveyor/storage furniture, all potentially animated
(`FPS`, `CIRCULAR`) and multi-directional (`ROTATES`).

Realistic options, cheapest to most expensive:

1. **Keep reusing vanilla furniture art** (what flax v1 does) — zero art work,
   the building just looks like a generic loom/refinery. Fine long-term if you
   don't mind visual reuse.
2. **Recolor existing frames** — duplicate the relevant sheet, tint the
   flax/hemp/wool/silk version's frames a different hue (e.g. give the Retting
   Shed a wet/green-brown palette vs. the Weaver's warm wood tones). Cheap,
   gives each building a distinct silhouette-adjacent identity without
   drawing new poses.
3. **Full custom spritesheet** — draw new frames matching the same grid
   layout as `REFINER.png`, register a *new* sheet path, and point your room's
   `SPRITES:` frame references at it instead of `REFINER: n`. This is real
   pixel-art production work (multiple rotations × animation frames per
   furniture piece) — budget for it accordingly, and only do it once the
   mechanics are fully proven.

## Order of operations recommendation

Given the project's "small confirmed loop" style: finish proving out *all
four* fiber chains mechanically with placeholder/recolored art first, then do
one dedicated art pass across all of them together. Doing real art per-material
as you go means re-learning the spritesheet conventions four times instead of
once, and risks throwing away art if a mechanical redesign changes which rooms
exist.
