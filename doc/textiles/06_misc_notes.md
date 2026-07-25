# Misc Notes

## Room filenames must start with a recognized type prefix

Room type is determined by filename prefix, matched against a hardcoded
list in `settlement.room.main.util.RoomsCreator` (`FARM_`, `REFINER_`,
`WORKSHOP_`, `PASTURE_`, `MINE_`, etc. — see
[doc/howto/make_custom_room.md](../howto/make_custom_room.md)).

- If a room file's key doesn't start with one of these prefixes, the game
  silently never constructs it — no error, not buildable, not in the build
  menu.
- If a room like that was meant to produce a resource, that resource ends
  up with zero registered producers.
- If a resource has zero producers, the game crashes
  (`NullPointerException` in `RecipeRatesVanilla`/`RecipeRates.bestRecipe()`)
  the moment a settlement is created or loaded.

Hit this with `SPINNER_COTTON.txt`/`SPINNER_WOOL.txt` (invented prefix, not
recognized). Fixed by renaming to `REFINER_`-prefixed names — mechanically
these are refiners (labor converts one resource to another), so `REFINER_`
is correct. (Later consolidated into a single `REFINER_SPINNER.txt` with
both recipes — see below — but the prefix fix itself is the point here.)

**Rule going forward: only use `FARM_`/`REFINER_`/`WORKSHOP_`/`PASTURE_`/
`MINE_` (etc. — check `make_custom_room.md` for the full list) as room-file
prefixes.** A genuinely new room type needs Java code (see that same doc).

## Consolidated processing rooms: one building, multiple recipes

Originally one Spinner/Weaver/Tailor room per material (`REFINER_SPINNER_COTTON`,
`REFINER_SPINNER_WOOL`, etc.). Consolidated into one `REFINER_SPINNER` (a
new sibling room), and the textile recipes for Weaver/Tailor were merged
directly into vanilla's own `REFINER_WEAVER`/`WORKSHOP_TAILOR` (see next
section) rather than kept as separate `_TEXTILES` rooms - fewer buttons in
the build menu, same pattern vanilla's own `WORKSHOP_TAILOR` already used
(3 recipes in one room) before the merge.

## Don't ship a parallel room next to a vanilla one if you can just overwrite it

`REFINER_WEAVER_TEXTILES` and `WORKSHOP_TAILOR_TEXTILES` were an earlier
attempt to avoid touching vanilla files - each shipped as a separate sibling
room next to the untouched vanilla `REFINER_WEAVER`/`WORKSHOP_TAILOR`. In
practice this meant two Weavers and two Tailors in the build menu, which
reads as a bug rather than a feature. Fixed by overwriting the vanilla
files directly (`__OVERWRITE: true`, full vanilla content copied in plus
the new recipes appended) and deleting the `_TEXTILES` rooms - see
[05](05_integration_and_tech.md#one-weaver-one-tailor-overwriting-the-vanilla-rooms).
This is different from the `PASTURE_ONX`/`PASTURE_ONX_WOOL` case, where the
sibling room is kept deliberately (the player needs both a food-focused and
a wool-focused herd option side by side, not a replacement).

## Husbandry category requires PASTURE_ + a real animal entity

Room category is hardcoded per room *type*, not configurable per-file -
confirmed via `ROOMS.java`: every `RoomsCreator<T>` call hardwires one
category to one type (`CATS.REFINERS` for all `REFINER_` rooms,
`CATS.CRAFTING` for all `WORKSHOP_` rooms, etc.). `PASTURE_` is the *only*
type mapped to `CATS.HUSBANDRY`. `ROOM_PASTURE.java` requires a valid
`ANIMAL:` key resolving to a registered `animal/*.txt` entity - no way
around this for a Husbandry-categorized room.

Used for the Silkworm Breeder: added a minimal `animal/SILKWORM.txt` purely
so `PASTURE_SILKWORM` qualifies for Husbandry. See below for why it's no
longer tiny/Onx-shaped.

**`PASTURE_` rooms cannot consume an `IN` resource - confirmed, not just
unverified.** Originally tried `IN: {MULBERRY: 3}` on `PASTURE_SILKWORM`,
reasoning that `Industry.java`'s `IN`/`OUT` parsing is shared code across
every room type so it should work the same everywhere. It doesn't: the game
fails to load with `Can't declare inputs to a pasture industry` - a
hardcoded restriction specific to the Pasture room type, not just an
untested pattern. `PASTURE_SILKWORM` is zero-input like every vanilla
pasture; `FARM_MULBERRY`/`MULBERRY` were removed since nothing can consume
them. If a Mulberry-feeding mechanic is wanted later, it needs a two-stage
split (a zero-input Husbandry pasture producing an intermediate "cocoon"
good, then a separate `REFINER_`/`WORKSHOP_` step consuming Mulberry +
cocoons) - not attempted.

## Silkworm reskinned as a "Silkcrawler" (Balticrawler-alike)

The original `animal/SILKWORM.txt` (`MASS: 1`, reused Onx sprite) looked
wrong in play: a tiny creature wandering the pasture like a full-size
animal. Worse, `MASS: 1` turned out to be actively counterproductive - the
pasture animal-density formula (`ROOM_PASTURE`/`PastureInstance`:
`CLAMP.d(2.5/(mass+10), 0, 1/9)`) clamps at `1/9 ≈ 0.111` animals/tile
regardless of how low `MASS` goes, so `MASS: 1` was already at the engine's
maximum possible density - `PASTURE_SILKWORM` had ~2.4x *more* wandering
creatures per tile than a same-size Onx or Globdien pen, the opposite of
"looks small and sparse."

Three fixes were investigated before landing on the current one:

- **Suppress the wandering AI / render as a swarm or count-only visual.**
  Not possible via config. Every pasture animal is spawned as a full,
  individually-pathfinding `Animal` entity running the same hardcoded
  `STAND`/`WALK_RANDOM`/`GRACE` states, rendered unconditionally every
  frame (`settlement.entity.animal.Animal`). `INDOORS: true` only affects
  room construction/fence/fertility, not animal spawning or movement.
  `PACK`/`GRAZES` are parsed (`AnimalSpecies.java`) but read nowhere else
  in the codebase - confirmed dead fields. Would need new Java overriding
  `PastureInstance`/`ROOM_PASTURE`.
- **Split into a small Husbandry breeder (producing an intermediate
  "cocoon" stock resource) + a Refiner "Sericulture House" consuming
  Mulberry + cocoons.** Verified mechanically sound: a Pasture's
  `INDUSTRY.OUT` production is scaled by `animalsCurrent/animalsMax` (a
  *fill ratio*, confirmed via `PastureInstance`'s `Animals`/`Adults`/
  `Tending` `RoomBoost`s), not raw headcount - a small, fully-stocked pen
  produces at full rate. This also would have cleanly revived `MULBERRY`
  (removed earlier - see above - only because Pasture can't have `IN`; a
  Refiner has no such restriction). Set aside in favor of the simpler,
  single-building fix below.
- **An Agriculture-categorized "Silk Farm" consuming Fruit + Silkworm
  stock via `IN`.** Confirmed **not achievable through config** for two
  independent reasons: `ROOM_FARM.java` hard-requires a `GROWABLE:` crop
  (`RESOURCES.growable().MAP.read(...)` - a non-optional read, throws if
  the key is missing), and `FarmInstance`/`Tile`/`Time` (the Farm room's
  actual per-tick production logic) never reads or consumes an `IN` block
  at all - that machinery lives entirely in the separate
  `settlement.room.industry.module`/`RoomProduction`/`RoomConsumption`
  classes that Refiner/Workshop rooms use. A Farm-type room *can*
  syntactically declare `IN` without an error (unlike Pasture, which
  explicitly rejects it in `ROOM_PASTURE.java`) - but it would just be
  silently dead config, never consumed. Category is also hardcoded per
  room-type-prefix in `ROOMS.java` (see above) - no way to get
  "Agriculture" + "Refiner-style IN/OUT" without writing a whole new room
  type in Java (the officially-documented extension path in
  `doc/howto/make_custom_room.md`).

**Landed on: reskin the animal as a "Silkcrawler," reusing vanilla
Balticrawler's stats/sprite wholesale.** Balticrawler is already the
closest vanilla precedent - its own flavor text calls it "huge larvae,"
and it's the *only* vanilla animal using `INDOORS: true`. Instead of
fighting the engine to make the creature look tiny, the fix leans into it
being a big, lore-abstracted creature - one visible Silkcrawler represents
a much larger farmed brood, so wandering behavior no longer reads as
incongruous, and `MASS: 200` drops the density to a normal pasture's
level (`2.5/210 ≈ 0.012`/tile, ~10x sparser than the old `MASS: 1`).

Two things were required and verified safe via the decompiled source
before copying Balticrawler's stats:
- **Non-edible, guaranteed.** A dead animal's resource drop is gated
  strictly on the animal's own `RESOURCES:` list (`Animal.java`'s
  natural-death path, `ThingsCadavers.java`), and a Pasture's slaughter
  payout is gated strictly on the room's own `INDUSTRIES[].OUT` block
  (`PastureInstance.slaughterAll()`) - neither path has a hardcoded `MEAT`
  fallback. Keeping `RESOURCES: [RAW_SILK,]` (never copying Balticrawler's
  `[MEAT,]`) and never adding `MEAT` to the room's `OUT` makes this
  structurally impossible to violate, independent of every other stat
  copied from Balticrawler.
- **Deliberately not copying Balticrawler's cave-dwelling side**
  (`LIVES_IN_CAVES: 0.8`, cave/mountain-weighted `TERRAIN`) - shelter
  comes from `PASTURE_SILKWORM`'s own `INDOORS: true` alone, matching real
  silk sheds, not caves.
- **Temperature sensitivity, modeled through the room, not the animal.**
  Real *Bombyx mori* cultivation has a narrow optimal band (~24-28°C),
  poor cold tolerance, and heat-stress/disease risk above ~30-35°C -
  historically a temperate-climate craft (China), not tropical or arctic.
  Confirmed the animal's own `CLIMATE:` key is mechanically inert for a
  tech-locked, never-wild-spawning species - `occurence(CLIMATE)` is only
  read by wild-spawn generation (`Animals.java`, `Generator.java`) and
  hunting yield (`WorkHunter.java`), none of which apply here. The real
  lever is `PASTURE_SILKWORM.txt`'s own `BONUS.CLIMATE`, which does affect
  production - retuned so `HOT` is a penalty (`0.5`) rather than the old
  bonus (`1.2`), `COLD` stays harsh (`0.15`), `TEMPERATE` is the clear
  peak (`1.0`).

## Dye chain placeholder art: Herb, not Cotton

`DYE_PLANT`/`DYE`/`FARM_DYE`/the `DYEF*` tech icons originally reused
vanilla's Cotton sprite/icon as a placeholder. Visually wrong (a dye plant
isn't a cotton boll) and confusing next to the real `COTTON` resource in
the UI. Switched to reusing Herb's sprite/icon instead
(`24->resource->Herb->0`, `SPRITE: Herb`), including the growable's
`STEM`/`GROWTH` tint values (copied from vanilla `HERB.txt` - those tints
are tuned for Herb's sprite geometry, so reusing Cotton's would look wrong
painted onto Herb's shape). Same reasoning `SILK00`/`PASTURE_SILKWORM`
already used reusing the Onx sprite for the Silkworm Breeder - pick the
closest-fitting vanilla placeholder, not just any placeholder, until real
art exists.

## Every custom room needs sound-key entries, or the game logs warnings on exit

Any room key without a matching entry in `assets/audio/config/ambience/Room.txt`
(`ROOM_<KEY>`) or `assets/audio/config/mono/AA_WORK.txt`
(`ROOM_WORK_<KEY>`/`ROOM_CLICK_<KEY>`) gets logged as "no ambiance sound by
the key of..."/"no race sound by the key of..." to
`~/.local/share/songsofsyx/logs/UnhandledDump.txt` on exit - not fatal, but
noisy, and the dump file accumulates across every past session rather than
resetting per run (so it can show room keys from long-deleted rooms too,
e.g. the old Flax rooms or the pre-merge `_TEXTILES`/`_COTTON`/`_WOOL`
rooms).

Vanilla's own convention, confirmed by grepping `data.zip`: `CLICK` is
always `DUMMY`; `WORK` depends on room-type prefix - `FARM_`/`PASTURE_`
use `impact->Dig*`, `REFINER_` uses `work->Machine*` (`WORKSHOP_TAILOR`
uses `work->Fabric*`, but that's vanilla-owned and already covered).
Ambiance is `DUMMY` for every farm/pasture/refiner/workshop in vanilla too
- none of them have a dedicated ambiance loop. Fixed by adding matching
entries for every currently-active custom room
(`FARM_DYE`/`PASTURE_ONX_WOOL`/`PASTURE_SILKWORM`/`REFINER_DYER`/
`REFINER_SPINNER`) and removing the stale Flax-era entries. `REFINER_WEAVER`
and `WORKSHOP_TAILOR` don't need entries since they're vanilla-keyed rooms
now (see the room-merge section above) - vanilla's own sound config already
covers them.

## Any new resource file is automatically tradable

Any file under `assets/init/resource/` that doesn't collide with a vanilla
filename is auto-tradable — no separate registration step. `resource/supply/`
is only for goods citizens directly consume as a need (food, drink, clothes,
opiates), not a trade whitelist.

## Vanilla `COTTON`'s display name was "Fibre"

Fixed via partial text override — `assets/text/resource/COTTON.txt` sets
just `NAME`/`NAMES` to "Cotton". No `__OVERWRITE: true` needed since no
other key is touched. Same pattern applies to any other vanilla
naming/text tweak.

## Dye: resolved as a single abstracted resource, not per-hue

"Dyed" is a quality tier per worn good (`CLOTHES`→`DYED_CLOTHES`,
`FINE_CLOTHES`→`DYED_FINE_CLOTHES`), not a color property — avoids the
fiber × hue combinatorial explosion. Hue variety later is reskin/art work
on these two resources, not new mechanical resources per color. Whether
`COLOR`/`TINT` sprite keys extend to resource piles is unanswered and no
longer relevant to the design.

## Hemp's cordage/rope idea is dormant

Hemp was folded into the collapsed **Plant Fibre** bucket alongside Cotton
(see [04](04_material_process_chains.md)), so the earlier idea of a
hemp-specific `ROPE`/`CORDAGE` side-recipe isn't implemented. Resurrect only
if Hemp is split back out into its own chain.

## Partial-override text files need `_JSON_ADD: true` to add into an existing nested key

Adding new tech node text (`text/tech/{REFINER,HUSB,AGRI,WORKSHOP}.txt`)
by writing `TECHS: { SPIN01: {...}, ... }` at the top level looked correct
but silently broke: every new tech node showed a fallback name like
"Husbandry 14:1" instead of "Basic Silkworm Breeding".

Cause, confirmed by reading `init.tech.TechTree`/`snake2d.util.file.Json`/
`JsonValue` in the decompiled game source: a top-level key that already
exists in the base file (vanilla's own `TECHS` object, in this case) gets
**wholesale-replaced** by a mod's overlay value by default
(`map.putReplace(...)`) - not recursively merged. Our `TECHS: {...}` block
was replacing vanilla's entire `TECHS` object with only our new nodes, so
`TechTree.java`'s lookup (`jText.json("TECHS")`) found an object missing
every vanilla key, `text` came back `null` for our nodes too, and
`TECH.java` fell back to `tree.name + " " + (col+1) + ":" + (row+1)` for
the display name. No error was raised because `null` text is a valid,
silently-handled case.

**Fix: add a top-level `_JSON_ADD: true,` key to the mod's file.** This
flag is read directly off the overlay file (`e.map.containsKey("_JSON_ADD")`
in `Json`'s constructor) and makes the merge recurse into an
already-existing key instead of replacing it, so our new `TECHS.SPIN01`
etc. get added alongside vanilla's existing `TECHS.BAKE01` etc. rather than
replacing the whole object. Needed on any partial-override file that adds
a new key *inside* an existing nested object the base file already
populates - not needed for a flat top-level key (like the `COTTON.txt`
`NAME`/`NAMES` fix, a plain scalar replace) or for a brand-new resource
file with no vanilla counterpart at all (nothing to merge against).

Side note found during this investigation: `__OVERWRITE: true`, used on
the *init* side of these same tech files (and on `REFINER_WEAVER.txt`/
`WORKSHOP_TAILOR.txt`), does not appear anywhere in the decompiled merge
logic - it's very likely inert (an unrecognized key, silently ignored).
Those files work correctly anyway because they supply *complete* content
for every top-level key that already exists in vanilla (`TREE`, `TECHS`,
`INDUSTRIES`, etc.), so the default wholesale-replace-per-key behavior
produces the intended full replacement regardless of whether `__OVERWRITE`
does anything. Left in place since it's harmless and documents intent, but
don't rely on it to mean anything to the engine.

## Editing a vanilla TREE grid safely

Every category's `TREE` grid has spacer columns that are blank across
*every* row (verified by dumping the full grid before editing). New tech
nodes were placed in those columns, within the existing row range - this
only changes the *value* of an already-blank array element, never adds a
new row key or changes an array's length. Still used `__OVERWRITE: true`
with the full vanilla file copied in for these specific files (not a
partial override) - the risk of a bad partial-merge into nested
`TREE`/`TECHS` content silently dropping the rest of the category's tech
tree was judged worse than owning the file. See
[05](05_integration_and_tech.md) for the exact placements.

## Still unverified in-game

- Whether the player can select between a room's multiple recipes
  (`WORKSHOP_TAILOR`, eight recipes; `REFINER_SPINNER`/`REFINER_WEAVER`,
  two-three each) — config parses and builds cleanly, in-game selection UI
  unverified.
- Whether the new tech nodes render correctly in the tree UI (safely blank
  columns, but genuinely untested placement).
- Whether `animal/ONX.txt`'s `RESOURCES` list (`COTTON`→`WOOL`, for
  wild-Onx hunting yield) can be partially overridden safely — not
  implemented, non-blocking (Wool's real supply is `PASTURE_ONX_WOOL`).
- Whether `REQUIRES` works on a room config to gate buildability — zero
  vanilla usage to confirm against (note: `UNLOCKS_FACTION` does gate
  buildability, see [05](05_integration_and_tech.md) - this is a different,
  still-unverified mechanism).
- Exact frame-slicing convention for multi-frame resource/icon
  spritesheets — inferred from pixel dimensions only. Not blocking so far
  since all new resources reuse vanilla sprites wholesale.

## Repo housekeeping

Mod fully renamed from "Example Mod" to "Textiles Mod": `pom.xml`
(`artifactId`, `mod.name`), repo directory, `.run/*.xml`, `.idea/compiler.xml`.
Requires an IntelliJ reload/resync if module identity changes again.
