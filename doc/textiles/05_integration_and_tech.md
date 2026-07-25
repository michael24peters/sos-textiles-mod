# Integration & Tech Tree

## One Weaver, one Tailor: overwriting the vanilla rooms

Earlier versions of this mod kept the textile recipes in separate sibling
rooms (`REFINER_WEAVER_TEXTILES`, `WORKSHOP_TAILOR_TEXTILES`) so the
vanilla `REFINER_WEAVER`/`WORKSHOP_TAILOR` files wouldn't need touching.
That left two Weavers and two Tailors in the build menu, which was
confusing rather than convenient. Both are now merged into the vanilla
files directly, using `__OVERWRITE: true` with the full vanilla content
copied in plus the textile recipes appended:

- **`REFINER_WEAVER.txt`**: vanilla's single `COTTON → FABRIC` recipe is
  replaced outright by the three textile recipes (`COTTON_THREAD`,
  `WOOL_THREAD`, `RAW_SILK`). Cotton now goes through the Spinner first
  like every other fiber - there's no more direct-to-fabric bootstrap
  path.
- **`WORKSHOP_TAILOR.txt`**: vanilla's three recipes (`LEATHER → CLOTHES`,
  `FABRIC → CLOTHES`, `LEATHER → ARMOUR_LEATHER`) are kept as-is at
  indices 0-2 so the existing `TAILc0`/`TAILc1`/`TAILc2` consumption
  boosts (which target those indices) stay correct. The five textile
  recipes are appended at indices 3-7.

Both rooms keep their single vanilla tech tree (`WEAV01`-`WEAV04` for the
Weaver, `TAIL01`-`TAIL04` for the Tailor) unchanged - no new tech nodes
were needed since the room keys didn't change, only their recipe lists.
The Dyeing tech line (`DYEC01`/`DYEC02`/`DYECc0` in `WORKSHOP.txt`, see
below) was retargeted from `ROOM_WORKSHOP_TAILOR_TEXTILES` to
`ROOM_WORKSHOP_TAILOR`, and its Excellence node's consumption-index
targets shifted from `_1`/`_3`/`_4` to `_4`/`_6`/`_7` to match the new
indices of the three dye-consuming recipes.

Note: `DYEC01`/`DYEC02` boost `ROOM_WORKSHOP_TAILOR>ADD`, which affects
the *whole* room's output, including the vanilla Leather/Fabric/Armour
recipes - there's no per-recipe output-boost mechanism in this game,
only per-recipe consumption reduction (which the Excellence node uses
correctly). This overlaps with what `TAIL01`-`TAIL04` already do to the
same room; harmless (bonuses just stack) but worth knowing if the numbers
look generous.

## Civic equip slots

`assets/init/stats/equip/civic/_FINE_CLOTHES.txt`:
```
RESOURCE: FINE_CLOTHES,
MAX_AMOUNT: 4,
WEAR_RATE: 0.15,
DEFAULT_TARGET: 0,
STANDING: {
	CITIZEN: 0.25,
	SLAVE: 0,
	NOBLE: 4.0,
	PRIO: 10,
},
```
Creates an `EQUIP_CIVIC_FINE_CLOTHES` race-stat key automatically (leading
`_` collapses per `doc/config/race.md`'s dynamic-EQUIP-stat rule).

`DYED_CLOTHES.txt`:
```
RESOURCE: DYED_CLOTHES,
MAX_AMOUNT: 6,
WEAR_RATE: 0.25,
DEFAULT_TARGET: 0,
STANDING: {
	CITIZEN: 2.0,
	SLAVE: 0.2,
	NOBLE: 2.5,
	PRIO: 9,
},
```

`DYED_FINE_CLOTHES.txt` has the steepest `STANDING`:
`{CITIZEN: 0.1, SLAVE: 0, NOBLE: 5.0, PRIO: 10}`.

## Tech tree: implemented

Corrects an earlier version of this doc, which claimed nothing gates room
buildability. **`UNLOCKS_FACTION: [ROOM_<KEY>]` (no `_UPGRADE_N` suffix)
does gate a room's base buildability** - confirmed via
`init.value.Lockers`/`RoomBlueprintImp.java`: every room registers itself
as `Lockable` under key `ROOM_<roomKey>`, and stays locked until some tech's
`UNLOCKS_FACTION` list references that exact key. If nothing references it,
it's unlocked from the start (the vanilla default for most rooms).
`MUSH00`/`HERB00`/`SPIC00`/`GLOB00`/`WARB00` all use this to gate their
rooms; `ROOM_REFINER_WEAVER_UPGRADE_1`-style keys (with a suffix) only gate
an upgrade tier, not the base room - that part of the earlier doc was
correct.

**Placement**: every existing `TREE` grid row in `AGRI.txt`/`HUSB.txt`/
`REFINER.txt`/`WORKSHOP.txt` has spacer columns that are blank across every
row (verified by dumping each grid) - new tech nodes were slotted into
those columns within the *existing* row range, so no row was extended and
no existing row's array length changed. `WORKSHOP.txt` additionally has a
fully-blank row (07) separating its two material groups, used for Dyeing.
Because this still touches deeply-nested content inside `TREE`/`TECHS`
(risk of a bad partial-merge silently dropping the rest of the category's
tech tree), these four files use **`__OVERWRITE: true` with the full
vanilla content copied in** rather than a partial override - correctness
over avoiding file ownership. `text/tech/*.txt` overrides for the new nodes
stay partial (just adding new keys, same as every other text override in
this mod).

Nodes added, each following the *exact* tier pattern of its category's
other lines (naming scheme, node count, and column shape all matched to
precedent rather than invented ad hoc):

**`REFINER.txt`** - matches `BAKE`/`BREW`/`COAL`/`SMEL`/`WEAV`: 6-tier
main line (Basic → Skilled → Improved [upgrade] → Profficient → Expert →
Advanced [upgrade]) plus one Excellence node in the adjacent column.
`Spinner` and `Dyer` got a dedicated 2-column block each (columns 15-16
and 11-12) instead of being squeezed into someone else's spare column:

| Tree | Column (main / excellence) | Nodes | Locked? |
|---|---|---|---|
| Spinner | 15 / 16 | `SPIN01`→`SPIN02`→`SPINu1`→`SPIN03`→`SPIN04`→`SPINu2`, `SPINc0` | No |
| Dyer | 11 / 12 | `DYER01`→`DYER02`→`DYERu1`→`DYER03`→`DYER04`→`DYERu2`, `DYERc0` | No |

`REFINER_SPINNER.txt` gained an `UPGRADES: [...]` block (3 tiers, mirroring
`REFINER_WEAVER`'s but with a 3-element `RESOURCE_MASK` since the Spinner
has no `MACHINERY` resource) so `SPINu1`/`SPINu2` have something to unlock.

Every `SPIN*`/`DYER*` node also got the 2-layer `ICON` block vanilla nodes
have (`BG: 32->REFINER->4` for the category graphic, `BG: 32->TECH->N` for
the tier badge) - missing at first, which left these nodes rendering
without the tier-badge layout every other tech uses. `32->REFINER->4` is
the same placeholder icon `REFINER_SPINNER.txt`/`REFINER_DYER.txt` already
use for their own room icon (borrowed from Weaver, since neither has
dedicated art yet), so the tech icons match their rooms exactly, the same
way `WEAV01`'s icon matches `REFINER_WEAVER`'s.

**`HUSB.txt`** - matches `GLOB`/`WARB` (Globdien/Warbeast) exactly: unlock
node with `ICON`, then Basic → Tools → Skilled → Profficient → Advanced
Tools → Expert (7 nodes total, same `LEVEL_MAX`/`COSTS` progression as
Globdien's):

| Tree | Column | Nodes | Locked? |
|---|---|---|---|
| Silkworm Breeder | 13 (after Warbeast) | `SILK00`→`SILK01`→`SILKt1`→`SILK02`→`SILK03`→`SILKt2`→`SILK04` | **Yes** (`SILK00` gates `ROOM_PASTURE_SILKWORM`, same as `GLOB00`/`ROOM_PASTURE_GLOBDIEN`) |

`SILK00` reuses the placeholder Onx sprite (`32->animal->Onx->0`) for its
`ICON`, matching the placeholder art already used in `animal/SILKWORM.txt`
and `PASTURE_SILKWORM.txt`.

**`AGRI.txt`** - matches `MUSH`/`HERB`/`SPIC` (the other locked-behind-
research farms) exactly: unlock node with `ICON`, then Basic → Tools →
Skilled → Profficient → Advanced Tools → Expert (7 nodes total, same
`LEVEL_MAX`/`COSTS` progression as Mushroom's). `Dye Farm` was originally
shipped unlocked with only 3 nodes (`DYEF01`→`DYEFt1`→`DYEF02`) -
inconsistent with the other specialty crops, which are all locked behind
research. Expanded to the full 7-node shape and gated behind a new
`DYEF00` unlock node, same pattern as Mushroom/Herb/Spices/Silkworm
Breeder:

| Tree | Column | Nodes | Locked? |
|---|---|---|---|
| Dye Farm | 13 (after Spices) | `DYEF00`→`DYEF01`→`DYEFt1`→`DYEF02`→`DYEF03`→`DYEFt2`→`DYEF04` | **Yes** (`DYEF00` gates `ROOM_FARM_DYE`, same as `MUSH00`/`ROOM_FARM_MUSHROOM`) |

`DYEF00`'s `ICON` reuses the placeholder Herb resource icon
(`24->resource->Herb->0`), matching the placeholder art used throughout the
Dye chain (see below) - same reasoning as `SILK00` reusing the placeholder
Onx sprite.

**`WORKSHOP.txt`** - doesn't use the REFINER 6-tier shape; the Tailor/
Carpenter lines mix in recipe-unlock nodes and most farms/workshops here
don't have dedicated building-upgrade tiers. `Dyeing` was kept at its
original 3-node shape since it wasn't called out as needing expansion (and
has no vanilla-precedent building-upgrade tier to add one against):

| File | Column used | Nodes | Locked? |
|---|---|---|---|
| `WORKSHOP.txt` | 4, rows 07-09 | `DYEC01`→`DYEC02`→`DYECc0` | No |

Tech naming follows vanilla's own spelling exactly, including its
"Profficient" typo (not "Proficient") - matching the convention other
categories already use beats correcting a typo in one corner of the tree.

Onx (`ONX_01`+) already has its own vanilla tech line boosting
`ROOM_PASTURE_ONX` - not touched, and no separate line was added for
`ROOM_PASTURE_ONX_WOOL`.

## What tech can and can't gate

`REQUIRES_TECH_LEVEL` only gates other tech nodes. `UNLOCKS_FACTION` with a
bare `ROOM_<KEY>` gates that room's buildability (see above);
`UNLOCKS_FACTION` with a `ROOM_<KEY>_UPGRADE_N` suffix only gates that
upgrade tier. `REQUIRES` on a room config itself is unused by every vanilla
room, still unverified whether it works.
