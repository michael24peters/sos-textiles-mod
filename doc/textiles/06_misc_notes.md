# Misc Notes

## Why LINEN was already tradable with zero extra steps

There is no separate "market registration" step in this game. Any file added
under `assets/init/resource/` that doesn't collide with a vanilla filename is
picked up automatically and is tradable by the same mechanism vanilla
`FABRIC` is — no `resource/supply/` entry needed for that (that folder is
only for the handful of goods citizens directly *consume* as a need — food,
drink, clothes, opiates — not a trade whitelist). This means "register X as
tradable" and "define resource X" are the same step, always. Worth remembering
so you don't go looking for a registration step that doesn't exist as you add
hemp/wool/silk.

## The "Fibre" naming fix

Vanilla `COTTON`'s in-game display name is literally "Fibre" (a generic
placeholder — `assets/text/resource/COTTON.txt`: `NAME: "Fibre"`). That was
harmless when cotton was the only fiber in the game, but became actively
confusing once Flax (correctly named "Flax") exists alongside a
generically-named "Fibre." Fixed via a partial override —
`src/main/resources/mod-files/assets/text/resource/COTTON.txt` now just
contains the three text keys with `NAME`/`NAMES` changed to "Cotton", no
`__OVERWRITE: true` needed since we're not touching any key beyond what the
vanilla file already had. This is the general pattern for any other vanilla
naming/text tweak you want later (e.g. if `FABRIC`'s description ever needs
updating to mention Linen/Wool Cloth as alternatives once those exist) —
partial text-file overrides are cheap and don't require replicating unrelated
vanilla content.

## Dyes will need upfront research before design starts

Two unresolved questions, deliberately deferred rather than guessed at:

1. **Does a "tinted resource variant" concept exist** (one `CLOTHES` resource
   with a dye-color property), or does every dye × cloth-type combination need
   to be its own named resource (`LINEN_RED`, `WOOL_BLUE`, ...)? The latter
   is combinatorial — 4 fibers × even 4 dyes is 16 resources, each needing its
   own art/icon/text/room-recipe wiring, before any of that art work is done.
   Check whether `COLOR`/`TINT` sprite keys (already used for room furniture
   recoloring per `doc/config/room.md`'s SPRITES key) extend to resource
   piles, or whether this needs a genuinely different mechanism.
2. Real historical dyeing (per the book) is itself a whole materials economy
   — mordants, specific plants/insects/shellfish per color, wildly different
   costs (woad/indigo vs. Tyrian purple). Decide up front whether dyes are a
   flavor layer (a handful of colors, roughly equal cost) or a second economic
   subsystem as deep as the fiber chains themselves — that's a scope decision
   worth making explicitly rather than discovering mid-build.

## Hemp's real-world dual-use as a hook, not just flavor

Worth treating as more than a reskin of flax: hemp's dominant historical
economic role (per the book) was cordage — ship rigging, rope — not cloth.
Mechanically, that's a second `INDUSTRIES` recipe on the same hemp-processing
room outputting a `ROPE`/`CORDAGE` resource instead of/alongside cloth. If the
game has any existing consumer for rope-like goods (shipbuilding, siege
equipment, `_MILITARY_SUPPLY`) that's a much more interesting integration
than "hemp is slightly cheaper cotton." Worth a scan of vanilla room
`RESOURCES` lists for anything already implying a cordage-shaped gap before
committing to this.

## Things not yet verified that the next session should test small before relying on

- Whether a new tech-tree `TREE` column / `TECHS` entries can be *added* into
  an existing vanilla category file (`AGRI.txt`, `REFINER.txt`) via partial
  override, or need full `__OVERWRITE`. (See
  [05_integration_and_tech.md](05_integration_and_tech.md).)
- Whether `REQUIRES` actually works on a room config at all, given zero
  vanilla usage to confirm against — if true tech-gated room *availability*
  (not just improvement) turns out to matter to you, test this in isolation
  first.
- Exact frame-slicing convention for multi-frame resource/icon spritesheets —
  currently inferred from pixel dimensions, not from documentation. (See
  [02_custom_art_guide.md](02_custom_art_guide.md).)

## Repo housekeeping from this session

- Mod renamed from "Example Mod" to "Textiles Mod" (`pom.xml` properties +
  `MainScript.java`'s `INFO` + log tags) — display name/install-folder name
  only, the Maven `artifactId`/module name (`songs-of-syx-mod-example`) and
  `.run/*.xml` configs were deliberately left untouched to avoid disrupting
  the already-working IntelliJ setup. Revisit if you want full consistency
  later — it's a bigger, riskier change (needs an IntelliJ Maven resync).
