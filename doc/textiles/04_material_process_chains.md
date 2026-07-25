# Material Process Chains

Three material lines - **Plant Fibre** (collapsed cotton/flax/hemp bucket),
**Wool**, **Silk** - plus a **Dye** layer across all three. Processing
(Spinner/Weaver/Tailor) is consolidated into one room per stage, each with
multiple recipes the player picks between - see
[06](06_misc_notes.md#consolidated-processing-rooms-one-building-multiple-recipes)
for why. Concrete proposed values are in [03_economic_levers.md](03_economic_levers.md).

## Two patterns for "multiple buildings for one stage"

**A) `UPGRADES[]` on one room file** - vanilla-idiomatic "better version of
the same building." Use by default.

**B) A wholly separate `ROOM_` file** - for a genuinely distinct building.
Used for `REFINER_SPINNER`, `REFINER_DYER`, `FARM_DYE`, `PASTURE_SILKWORM`,
and `PASTURE_ONX_WOOL` (a sibling to vanilla `PASTURE_ONX`, not a
replacement - see below).

**Weaver and Tailor are the exception**: rather than a third sibling room,
`REFINER_WEAVER.txt`/`WORKSHOP_TAILOR.txt` are fully overwritten
(`__OVERWRITE: true`) with the textile recipes appended to vanilla's own.
The mod originally shipped these as separate `_TEXTILES` rooms to avoid
touching vanilla files, but that left two Weavers and two Tailors in the
build menu - overwriting is the correct fix. See
[05](05_integration_and_tech.md) for exactly what changed.

`UNLOCKS_FACTION: [ROOM_<KEY>]` (no `_UPGRADE_N` suffix) gates a room's base
buildability - confirmed via source, see [06](06_misc_notes.md). Used for
Silkworm Breeder and Dye Farm (matching vanilla's own pattern of locking
specialty crops/pastures - Mushroom, Herb, Poppy, Globdien, Warbeast - behind
research); everything else is buildable from the start.

## Raw production (per material)

- **Plant Fibre**: `FARM_COTTON` (vanilla, unmodified) → `COTTON`.
- **Wool**: `PASTURE_ONX_WOOL` (new sibling room to vanilla `PASTURE_ONX`,
  which stays untouched) → `WOOL`. Two rooms sharing `ANIMAL: ONX` because
  there's no in-vanilla mechanic for one pasture to switch between a
  food-focused and wool-focused herd - see [06](06_misc_notes.md).
- **Silk**: `PASTURE_SILKWORM` → `RAW_SILK`, zero-input like every vanilla
  pasture (the engine does not support `IN` on a pasture's industry -
  confirmed by a crash, see [06](06_misc_notes.md)). Originally planned to
  consume `MULBERRY` from a `FARM_MULBERRY`; both were removed once that
  turned out to be unsupported. `PASTURE_SILKWORM` (Husbandry category, not
  Refining) requires a registered `animal/SILKWORM.txt` entity - `PASTURE_`
  is the only room type mapped to Husbandry. That animal ("Silkcrawler" in
  its own text) reuses vanilla Balticrawler's stats/sprite wholesale rather
  than being genuinely tiny - see [06](06_misc_notes.md) for why. Gated
  behind research (`SILK00` tech, matching Mushroom/Herb/Globdien/
  Warbeast's pattern) - see [05](05_integration_and_tech.md).
- **Dye**: `FARM_DYE` → `DYE_PLANT`.

## Consolidated processing rooms

**`REFINER_SPINNER`** (one building, two recipes):
- `COTTON → COTTON_THREAD`
- `WOOL → WOOL_THREAD`

No recipe for Silk - silk fiber is reeled off the cocoon as a continuous
filament, not spun from staple fibers.

**`REFINER_WEAVER`** (overwrites vanilla; one building, three recipes -
vanilla's direct `COTTON → FABRIC` recipe is gone, Cotton now goes through
the Spinner like every other fiber):
- `COTTON_THREAD → FABRIC` (`+10%` better ratio than vanilla's old flat `2:2`)
- `WOOL_THREAD → WOOL_FABRIC`
- `RAW_SILK → SILK`

**`WORKSHOP_TAILOR`** (overwrites vanilla; one building, eight recipes -
vanilla's three (`LEATHER → CLOTHES`, `FABRIC → CLOTHES`,
`LEATHER → ARMOUR_LEATHER`) stay untouched at indices 0-2, five textile
recipes appended at indices 3-7):
- `WOOL_FABRIC → CLOTHES`
- `WOOL_FABRIC + DYE → DYED_CLOTHES`
- `SILK → FINE_CLOTHES`
- `SILK + DYE → DYED_FINE_CLOTHES`
- `FABRIC + DYE → DYED_CLOTHES` (the Plant Fibre path's dyed variant)

**`REFINER_DYER`** (one recipe): `DYE_PLANT → DYE`.

## Civic equip slots

- `FINE_CLOTHES` feeds a new equip slot (`_FINE_CLOTHES`), not plain
  `CLOTHES`, weighted toward `NOBLE`.
- `DYED_CLOTHES`/`DYED_FINE_CLOTHES` each feed their own equip slot.

See [05](05_integration_and_tech.md).
