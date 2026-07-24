# Material Process Chains

## Two patterns for "multiple buildings for one stage"

Before the per-material lists: there are two different ways to have more than
one building for the same processing stage, and they mean different things.

**A) `UPGRADES[]` on one room file** — the vanilla-idiomatic way to represent
"a better version of the same building." Same room, same footprint, tech
unlocks the next `UPGRADES` tier (`UNLOCKS_FACTION: [ROOM_X_UPGRADE_N]`), cost
scales via `RESOURCE_MASK`. Use this by default — it's what nearly every
vanilla refiner/workshop/pasture does.

**B) A wholly separate `ROOM_` file** — a genuinely distinct building (own
name, icon, footprint, art) that coexists on the map alongside the basic one,
the way `doc/howto/make_custom_room.md`'s `WORKSHOP2` example is a sibling to
`WORKSHOP_BOWYER` rather than an upgrade of it. Use this only when the fantasy
really calls for a visually/thematically distinct structure (e.g. a manual
"Retting Shed" vs. a later-game "Water-Powered Fulling Mill" that looks and
sounds different, not just numerically better) — it's more art and text work
per tier, so don't reach for it by default.

One honest caveat: nothing in the vanilla game actually **gates the
buildability** of a base industry room behind a researched technology — a
`grep` across every vanilla `room/*.txt` file turns up zero uses of the
documented `REQUIRES` key. Tech in this game *improves* rooms you can already
build (output/efficiency `BOOST`s) and *unlocks upgrade tiers* within a room
(`UNLOCKS_FACTION`); it doesn't lock the base Cotton Farm behind a "Cotton
Farming I" research the way a 4X game might. If you want true "can't build
this until researched" gating for a new material, that's an experimental
deviation from how the game normally works, not a documented/proven pattern —
test it carefully in isolation before relying on it. See
[05_integration_and_tech.md](05_integration_and_tech.md).

---

## Cotton (vanilla, reference)

1. `FARM_COTTON` — grows `COTTON`
2. `REFINER_WEAVER` — `COTTON` → `FABRIC`
3. `WORKSHOP_TAILOR` — `FABRIC` or `LEATHER` → `CLOTHES`

## Flax → Linen (v1 built, stops at step 2)

1. `FARM_FLAX` — grows `FLAX` ✅ built
2. `REFINER_RETTER_FLAX` ("Retting Shed") — `FLAX` → `LINEN` ✅ built
3. *(not yet built)* feed `LINEN` into `CLOTHES` production — see
   [05](05_integration_and_tech.md) for how to do this without editing the
   vanilla Tailor file

**Optional v2 deepening**: split step 2 into two rooms for more depth, closer
to the real retting→spinning→weaving process and matching your original
"retting sheds" + "weaving chain" framing:
- `REFINER_RETTER_FLAX` — `FLAX` → `FLAX_FIBER` (retting + scutching)
- `REFINER_WEAVER_LINEN` — `FLAX_FIBER` → `LINEN` (spinning + weaving)

Not necessary for the mechanics to work (v1's single-step version is exactly
as valid as cotton's own single-step abstraction) — purely a depth/flavor
call once the basics are proven.

## Hemp → Hemp Cloth (not yet built; near-identical to flax)

1. `FARM_HEMP` — grows `HEMP`, broad climate tolerance, fast `GROWTH_VALUE`
2. `REFINER_RETTER_HEMP` — `HEMP` → `HEMP_CLOTH`
3. feed into `CLOTHES`, same as linen

Building this is close to a search-and-replace of the flax files. The one
thing worth doing differently: real hemp's dominant historical use was
cordage/rigging, not cloth (see *The Fabric of Civilization*'s treatment of
it) — consider giving hemp a *second* output path to a new `ROPE`/`CORDAGE`
resource via a second `INDUSTRIES` recipe on the same refiner room (mirroring
how `WORKSHOP_TAILOR` already has multiple unrelated recipes side by side).
This gives hemp a genuine mechanical identity instead of being "cheap flax."
See [06_misc_notes.md](06_misc_notes.md).

## Wool (not yet built; needs one new design decision)

1. **New `ANIMAL`**: `assets/init/animal/SHEEP.txt` — for a first pass, point
   `SPRITE` at an existing vanilla animal (e.g. Auroch or Mount) as a
   placeholder; animal sprites are walk-cycle sheets, much more art-intensive
   than a resource icon.
2. `PASTURE_SHEEP` — template: `PASTURE_AUR.txt`. `ANIMAL: SHEEP`,
   `INDUSTRIES.INDUSTRY.OUT` produces `WOOL` (and optionally `MEAT`/`LIVESTOCK`
   alongside, mirroring Auroch's multi-output pattern — real sheep also
   produce meat).
3. `REFINER_FULLER` ("Woolen Mill" / "Fulling Mill") — `WOOL` → `WOOL_CLOTH`
4. feed into `CLOTHES`; consider leaning `WOOL_CLOTH`'s eventual civic-equip
   `BOOST` harder into `PHYSICS_RESISTANCE_COLD` than linen/cotton, since
   that's wool's real-world distinguishing property.

## Silk (not yet built; needs one new design decision)

Rejected approach: a literal silkworm-breeding room modeled on
`BREEDER_GARTHIMI.txt`. That room is a hardcoded race-population mechanic
(`RACE:`, `INCUBATION_DAYS:`), not a generic resource-output breeder — it
doesn't generalize to insects producing a tradable resource.

Recommended approach instead:

1. `FARM_MULBERRY` — grows `MULBERRY` (leaves), template: any `FARM_` room
2. `REFINER_SERICULTURE` ("Sericulture House") — `MULBERRY` → `RAW_SILK`,
   abstracting silkworm-rearing/cocoon-harvesting the same way `FARM_COTTON`
   abstracts ginning
3. `REFINER_SILK_WEAVER` — `RAW_SILK` → `SILK`
4. Feed `SILK` into a **new** civic equip slot (e.g. `_FINE_CLOTHES`) rather
   than plain `CLOTHES`, weighted hard toward `NOBLE` in `STANDING` — this is
   where silk's "for nobility" identity actually gets enforced mechanically.
   See [05](05_integration_and_tech.md).

Keep silk's output rate low and tech cost high (per
[03_economic_levers.md](03_economic_levers.md)) so scarcity is real, not just
themed.

## Dyes (not yet researched — deferred until the four fiber chains work end to end)

Two open questions to resolve before designing this, flagged in
[06_misc_notes.md](06_misc_notes.md): whether the engine supports resource
"variants" (a tinted `CLOTHES` per dye) at the recipe level, or whether each
dye × cloth combination needs to be its own resource (combinatorial explosion
with 4+ fibers × several dyes). Investigate before committing to a design.
