# Implementation Log

A running record of what was actually built, next to what was originally
proposed. Update whenever a number gets retuned after playtesting - don't
pre-fill "As implemented" with guesses, leave it as "not yet playtested"
until it's actually been played.

## Fixed: crash on settlement creation

Cause and fix: [06_misc_notes.md](06_misc_notes.md#room-filenames-must-start-with-a-recognized-type-prefix)
(`SPINNER_COTTON`/`SPINNER_WOOL` used an unrecognized room prefix).
Confirmed fixed by playtest.

## Consolidated into one Spinner, one Weaver, one Tailor

Per-material rooms (`REFINER_SPINNER_COTTON`/`_WOOL`,
`REFINER_WEAVER_COTTON`/`_WOOL`, `REFINER_SILK_WEAVER`,
`WORKSHOP_TAILOR_WOOL`/`_SILK`/`_DYE`) replaced by three rooms, each with
multiple recipes: `REFINER_SPINNER`, `REFINER_WEAVER`,
`WORKSHOP_TAILOR`. See [04](04_material_process_chains.md) for the
recipe list and [06](06_misc_notes.md) for why.

## Silkworm Breeder: recategorized to Husbandry, then found Pastures can't have inputs

`REFINER_SERICULTURE` replaced by `PASTURE_SILKWORM` (new `animal/SILKWORM.txt`
required - see [06](06_misc_notes.md)). First attempt kept the
`MULBERRY:3 → RAW_SILK:1` ratio as an `IN`/`OUT` pair - **crashed on load**:
`Can't declare inputs to a pasture industry`. Fixed by making
`PASTURE_SILKWORM` zero-input (`OUT: {RAW_SILK: {PLAYER: 0.5, ...}}` only,
matching every vanilla pasture) and removing `FARM_MULBERRY`/`MULBERRY`
entirely, since nothing can consume them. Gated behind research (`SILK00`
tech) - see [05](05_integration_and_tech.md). Confirmed fixed by playtest
(build succeeds; in-game verification pending).

## Plant Fibre

| Resource/Room | Field | Proposed | As implemented | Changed? Why |
|---|---|---|---|---|
| `COTTON_THREAD` | `DEGRADE_RATE` | `0.05` | `0.05` | No |
| `REFINER_SPINNER` (Cotton recipe) | `IN/OUT` | `COTTON:2 → COTTON_THREAD:2` | same | No |
| `REFINER_WEAVER` (Cotton Thread recipe) | `IN/OUT` | `COTTON_THREAD:2 → FABRIC:2.2` | same | Not yet playtested |

## Wool

| Resource/Room | Field | Proposed | As implemented | Changed? Why |
|---|---|---|---|---|
| `WOOL` | `DEGRADE_RATE` | `0.05` | `0.05` | No |
| `PASTURE_ONX_WOOL` | `OUT.WOOL` | v1 `{PLAYER: 3.0, AI_RATE: 3.4}` (matched `FARM_COTTON`) → v2 `{PLAYER: 1.8, AI_RATE: 3.6}` (matched Onx's own fiber rate) → v3 `{PLAYER: 3.0, AI_RATE: 6.0}` (Merino-style breed specialization) | `{PLAYER: 2.5, AI_RATE: 5.0, AI_RECOVERY: 0.5}` (v4) | **Yes, three times.** v3's Merino comparison was anachronistic for this mod's Bronze/Iron Age setting - real ancient wool-breeding (the "Secondary Products Revolution", ~4000-3000 BCE) was a real but far more modest specialization than any modern breed. `2.5` (`1.35x` over Onx's baseline `1.8`) is cross-checked against the game's full staple-vs-rarity spread. Full reasoning in [03](03_economic_levers.md). |
| `PASTURE_ONX_WOOL` | `OUT.MEAT` | `{PLAYER: 0.1, AI_RATE: 0.16}` | same | Not yet playtested |
| `PASTURE_ONX_WOOL` | `OUT.LIVESTOCK` | unchanged from vanilla `PASTURE_ONX` | same | No |
| `animal/ONX.txt` hunting-yield override | `RESOURCES` | `[MEAT, WOOL,]` | **not implemented** | Deferred, non-blocking |
| `REFINER_SPINNER` (Wool recipe) | `IN/OUT` | `WOOL:2 → WOOL_THREAD:2` | same | No |
| `REFINER_WEAVER` (Wool Thread recipe) | `IN/OUT` | `WOOL_THREAD:2 → WOOL_FABRIC:2.2` | same | Not yet playtested |
| `WORKSHOP_TAILOR` (Wool Fabric recipe) | `IN/OUT` | `WOOL_FABRIC:4.0 → CLOTHES:{PLAYER:3.0}` | same | No |

## Silk

| Resource/Room | Field | Proposed | As implemented | Changed? Why |
|---|---|---|---|---|
| `MULBERRY`, `FARM_MULBERRY` | - | planned | **removed** | Pastures can't have `IN` (confirmed by crash) - nothing left to consume Mulberry, see above |
| `PASTURE_SILKWORM` | `OUT` | `MULBERRY:3 → RAW_SILK:{PLAYER:1,AI_RATE:1}` | `RAW_SILK:{PLAYER:0.5,AI_RATE:0.5,AI_RECOVERY:0.5}`, zero-input | **Yes** - forced by the Pasture `IN` restriction; rate halved from the old net effective yield to keep scarcity without the conversion-loss lever, not yet calibrated |
| `REFINER_WEAVER` (Raw Silk recipe) | `IN/OUT` | `RAW_SILK:2 → SILK:2` | same | No |
| `WORKSHOP_TAILOR` (Silk recipe) | `IN/OUT` | `SILK:4 → FINE_CLOTHES:{PLAYER:1.5}` | same | Not yet playtested |
| `_FINE_CLOTHES` equip | `RESOURCE` | `FINE_CLOTHES` | same | No |
| `_FINE_CLOTHES` equip | `STANDING` | `{CITIZEN: 0.25, SLAVE: 0, NOBLE: 4.0, PRIO: 10}` | same | No |

## Dye

| Resource/Room | Field | Proposed | As implemented | Changed? Why |
|---|---|---|---|---|
| `DYE_PLANT` growable | `CLIMATE_BONUS` | `{COLD: 0.5, TEMPERATE: 1.0, HOT: 0.8}` | same | Not yet playtested |
| `FARM_DYE` | `OUT.DYE_PLANT` | `{PLAYER: 2.5, AI_RATE: 2.4}` | same | Not yet playtested |
| `REFINER_DYER` | `IN/OUT` | `DYE_PLANT:3 → DYE:1` | same | Not yet playtested |
| `WORKSHOP_TAILOR` (Wool dyed recipe) | `IN/OUT` | `WOOL_FABRIC:4.0 + DYE:1.0 → DYED_CLOTHES:{PLAYER:3.0}` | same | No |
| `WORKSHOP_TAILOR` (Silk dyed recipe) | `IN/OUT` | `SILK:4 + DYE:1.0 → DYED_FINE_CLOTHES:{PLAYER:1.5}` | same | No |
| `WORKSHOP_TAILOR` (Fabric dyed recipe) | `IN/OUT` | `FABRIC:4.0 + DYE:1.0 → DYED_CLOTHES:{PLAYER:3.0}` | same | No |
| `DYED_CLOTHES` equip | `STANDING` | `{CITIZEN: 2.0, SLAVE: 0.2, NOBLE: 2.5, PRIO: 9}` | same | Not yet playtested |

## Tech tree: implemented

Reverses the earlier "deferred" decision - see
[05_integration_and_tech.md](05_integration_and_tech.md) for the full node
lists, grid placement, and the `UNLOCKS_FACTION` gating correction. All 5
lines (`REFINER.txt` Spinner + Dyer, `HUSB.txt` Silkworm Breeder,
`AGRI.txt` Dye Farm, `WORKSHOP.txt` Dyeing) built and installed; not yet
playtested in-game.

## Tech tree: Spinner/Dyer expanded to 6 tiers, Silkworm Breeder to 7 nodes

First pass shipped Spinner and Dyer as short 3-4 node trees and Silkworm
Breeder as a 3-node one - inconsistent with every other line in their
categories. Fixed: Spinner and Dyer now match `REFINER.txt`'s own
`WEAV01`-`WEAVc0` shape exactly (Basic → Skilled → Improved [upgrade] →
Profficient → Expert → Advanced [upgrade], plus one Excellence node), and
Silkworm Breeder now matches `HUSB.txt`'s `GLOB00`-`GLOB04` shape exactly
(unlock → Basic → Tools → Skilled → Profficient → Advanced Tools →
Expert). `REFINER_SPINNER.txt` gained an `UPGRADES: [...]` block so its two
new building-upgrade nodes have something to unlock. Naming follows
vanilla's own spelling, including the "Profficient" typo, for consistency
with every other category. Full detail in
[05](05_integration_and_tech.md#tech-tree-implemented).

## Weaver and Tailor merged into the vanilla rooms

The `_TEXTILES` sibling-room pattern (kept to avoid touching vanilla files)
resulted in two Weavers and two Tailors in the build menu. Fixed by
overwriting `REFINER_WEAVER.txt`/`WORKSHOP_TAILOR.txt` directly
(`__OVERWRITE: true`) with the textile recipes merged into vanilla's own,
and deleting `REFINER_WEAVER_TEXTILES`/`WORKSHOP_TAILOR_TEXTILES`. Both
rooms keep their single vanilla tech tree unchanged. Full detail in
[05](05_integration_and_tech.md#one-weaver-one-tailor-overwriting-the-vanilla-rooms).

## Fixed: missing/stale room sound-key entries

`assets/audio/config/ambience/Room.txt` and
`assets/audio/config/mono/AA_WORK.txt` still had entries for the Flax-era
rooms (deleted earlier this session) and none for the current custom rooms,
producing "no ambiance sound"/"no race sound" warnings on every exit.
Fixed by removing the stale entries and adding matching ones for
`FARM_DYE`/`PASTURE_ONX_WOOL`/`PASTURE_SILKWORM`/`REFINER_DYER`/
`REFINER_SPINNER`, following vanilla's own per-room-type convention. Full
detail in
[06](06_misc_notes.md#every-custom-room-needs-sound-key-entries-or-the-game-logs-warnings-on-exit).

## Fixed: new tech nodes showed fallback names like "Husbandry 14:1"

All new tech text (`text/tech/{REFINER,HUSB,AGRI,WORKSHOP}.txt`) put node
entries at the top level (`SPIN01: {...}`) instead of nested under `TECHS`
like vanilla's own text files do. Without that nesting, the game's text
lookup couldn't find them and fell back to a coordinate-based name -
worse, the flat `TECHS: {...}` block that WAS added (once nesting was
fixed) would have silently wiped vanilla's own tech names in the same
category, since a partial override replaces an existing top-level key
wholesale by default. Fixed by nesting everything under `TECHS` and adding
a top-level `_JSON_ADD: true,` marker so the merge adds into vanilla's
`TECHS` object instead of replacing it. Confirmed by reading the
decompiled game source (`TechTree.java`, `Json.java`, `JsonValue.java`);
full detail in
[06](06_misc_notes.md#partial-override-text-files-need-_json_add-true-to-add-into-an-existing-nested-key).
Build succeeds; in-game verification pending.

## Dye Farm tech expanded to 7 nodes and locked behind research

Shipped with only 3 nodes and unlocked from the start - inconsistent with
Mushroom/Herb/Spices (the other specialty `AGRI` crops), which are all
7-node locked lines. Expanded `DYEF01`→`DYEFt1`→`DYEF02` to the full
`DYEF00`(unlock)→`DYEF01`→`DYEFt1`→`DYEF02`→`DYEF03`→`DYEFt2`→`DYEF04`
shape, matching Mushroom's exact `LEVEL_MAX`/`COSTS` progression, and
gated `ROOM_FARM_DYE` behind `DYEF00` the same way `MUSH00` gates
`ROOM_FARM_MUSHROOM`. Also added the missing `ICON` blocks to
`SPIN01`-`SPINu2`/`DYER01`-`DYERu2` (Refining category) - these had no
icon at all, unlike every vanilla tech node. Full detail in
[05](05_integration_and_tech.md#tech-tree-implemented).

## Dye chain art swapped from Cotton to Herb placeholder

`DYE_PLANT`/`DYE`/`FARM_DYE`/`DYEF*` tech icons reused Cotton's
sprite/icon, which looked wrong (not a dye plant) and confusable with the
real Cotton resource. Switched to Herb's sprite/icon throughout - full
detail in
[06](06_misc_notes.md#dye-chain-placeholder-art-herb-not-cotton).

## Silkworm reskinned as a "Silkcrawler" (Balticrawler-alike)

`animal/SILKWORM.txt`'s `MASS: 1` looked like a fix for "should look
small," but was actually the densest population the engine's pasture
formula allows (clamped at `1/9` animals/tile regardless of how low `MASS`
goes) - `PASTURE_SILKWORM` had more wandering creatures per tile than any
vanilla pasture, the opposite of the intended effect. Investigated three
fixes (suppressing the wandering AI entirely, a two-building Husbandry-
breeder-plus-Refiner-processor split reviving `MULBERRY`, and an
Agriculture-categorized farm consuming `IN`) - the first and third are
confirmed not achievable through config alone (would need new Java room-
type code), the second was mechanically sound but set aside for the
simpler option. Landed on reusing vanilla Balticrawler's stats/sprite
wholesale (`MASS: 1→200`, `HEIGHT: 1→5`, `DAMAGE.PIERCE: 0.01→0.2`,
`RESOURCE_AMOUNT: 0.1→0.5`, sprite `Onx→Balticrawler`) except its cave-
dwelling traits (kept `LIVES_IN_CAVES: 0`, `TERRAIN: {NONE: 1.0}` -
shelter comes from `PASTURE_SILKWORM`'s own `INDOORS: true`) and its
`RESOURCES` list (kept `RAW_SILK`, never `MEAT` - confirmed via source
that a Pasture's slaughter payout and an animal's natural-death drop are
both gated strictly on config, with no hardcoded `MEAT` fallback, so
Silkcrawlers cannot be eaten regardless of other stats copied). Also
retuned `PASTURE_SILKWORM.txt`'s `BONUS.CLIMATE` (`HOT: 1.2→0.5`,
`COLD: 0.1→0.15`) to model real *Bombyx mori* temperature sensitivity -
the animal's own `CLIMATE:` key is confirmed inert for a never-wild-
spawning species. Full reasoning in
[06](06_misc_notes.md#silkworm-reskinned-as-a-silkcrawler-balticrawler-alike).
Build succeeds; in-game verification pending (blocked on reaching the
relevant tech/resources in a live save).

## Open questions

See [06_misc_notes.md](06_misc_notes.md#still-unverified-in-game).
