# Economic Levers

Every config key that changes cost, price, labor, growth time, research, or
happiness/class impact, plus the concrete values chosen for Plant
Fibre/Wool/Silk/Dye. See [07_implementation_log.md](07_implementation_log.md)
for whether these survived playtesting.

## Per-resource levers (`assets/init/resource/<X>.txt`)

| Key | Effect |
|---|---|
| `DEGRADE_RATE` | Spoilage rate (0.0-1.0). Higher = more storage loss, pushes toward just-in-time production. |
| `PRICE_CAP` | Buy-price multiplier cap. Not set on any vanilla textile resource - engine default applies unless overridden. |
| `PRICE_MUL` | Sell-price multiplier. Same - not used by vanilla textile resources. |
| `CATEGORY_DEFAULT` | UI sort order only, cosmetic. |

## Per-growable levers (`assets/init/resource/growable/<X>.txt`)

| Key | Effect |
|---|---|
| `GROWTH_VALUE` | Growth speed. |
| `SEASONAL_OFFSET` | Shifts the growing season. |
| `CLIMATE_BONUS.{COLD,TEMPERATE,HOT}` | Where the plant thrives - strongest differentiation lever between materials. |

## Per-room levers (any `FARM_`/`PASTURE_`/`REFINER_`/`WORKSHOP_`)

| Key | Effect |
|---|---|
| `RESOURCES` + `AREA_COSTS` | Build materials and per-tile cost. |
| `ITEMS[].COSTS` | Per-furniture-piece build cost. |
| `BONUS.CLIMATE` | Production multiplier by climate, building level. Only appears on outdoor `FARM_`/`PASTURE_` rooms in vanilla, never on `REFINER_`/`WORKSHOP_`. |
| `WORK.SHIFT_OFFSET` / `FULFILLMENT` / `ACCIDENTS_PER_YEAR` / `NIGHT_SHIFT` / `HEALTH_FACTOR` | Labor conditions: morale, injury risk, night operation. |
| `INDUSTRY(.IES).OUT.<RES>.PLAYER` / `.AI_RATE` / `.AI_RECOVERY` | Production rate, separate for player vs. AI. |
| `INDUSTRIES.INDUSTRY.IN`/`OUT` ratio | Recipe efficiency/conversion loss. |
| `UPGRADES[].RESOURCE_MASK` / `.BOOST` / `.AI` | Progressive upgrade tiers. |
| `STORAGE` | (Refiner/Workshop only) buffer capacity. |
| `ANIMAL` | (Pasture only) required, must reference a registered `animal/*.txt` entity. `PASTURE_` is the only room type mapped to the Husbandry category - see [06](06_misc_notes.md). |

## Class-interplay levers (`assets/init/stats/equip/civic/<X>.txt`)

| Key | Effect |
|---|---|
| `RESOURCE` | Resource this equip slot consumes to "wear." |
| `MAX_AMOUNT` | Equip-level cap. |
| `WEAR_RATE` | Consumption velocity. |
| `DEFAULT_TARGET` | Starting demand level. |
| `BOOST.<KEY>` | Side-effects of wearing it - vanilla `CLOTHES` grants `PHYSICS_RESISTANCE_HOT`/`_COLD`. |
| `STANDING.{CHILD,CITIZEN,SLAVE,NOBLE}` | How much each class cares about this good for happiness/standing. |
| `EXPO` / `EXPONENT`, `ARRIVAL_AMOUNT` | Present in vanilla `_CLOTHES.txt`/`JEWELRY.txt`, not documented in `doc/config/`. `ARRIVAL_AMOUNT` is **required** - omitting it crashes the game on load. Always set both. |

## Tech-tree levers (`assets/init/tech/<CATEGORY>.txt`)

| Key | Effect |
|---|---|
| `COSTS.CIVIC_INNOVATION`/`CIVIC_KNOWLEDGE` | Research cost. |
| `LEVEL_MAX` | Levelable tiers, each reapplying its `BOOST`. |
| `REQUIRES_TECH_LEVEL` | Prerequisite chain among tech nodes. |
| `BOOST.ROOM_<KEY>>ADD` | Output-rate boost for that room, auto-registered with zero setup. |
| `BOOST.ROOM_CONSUMPTION_<KEY>_<i>>ADD` | Efficiency boost for recipe index `i` in that room. |
| `UNLOCKS_FACTION: [ROOM_<KEY>]` | **Gates that room's base buildability** until researched (no vanilla precedent for this in this mod's earlier docs - corrected, see [05](05_integration_and_tech.md)). |
| `UNLOCKS_FACTION: [ROOM_<KEY>_UPGRADE_N]` | Gates only upgrade tier `N`, not the base room. |

## Verified vanilla reference values

| File | Key values |
|---|---|
| `resource/COTTON.txt` | `DEGRADE_RATE: 0.05` |
| `resource/FABRIC.txt` | `DEGRADE_RATE: 0.05` |
| `resource/CLOTHES.txt` | `DEGRADE_RATE: 0.1` |
| `resource/growable/COTTON.txt` | `GROWTH_VALUE: 0.35`, `CLIMATE_BONUS: {COLD: 0.2, TEMPERATE: 0.8, HOT: 1.1}` |
| `room/FARM_COTTON.txt` | `INDUSTRY.OUT.COTTON: {PLAYER: 3, AI_RATE: 2.86}`; `BONUS.CLIMATE: {COLD: 0.5, TEMPERATE: 1, HOT: 1.5}` |
| `room/PASTURE_ONX.txt` | `INDUSTRY.OUT: {MEAT: 0.7, COTTON: 1.8, LIVESTOCK: 0.0875}`; `BONUS.CLIMATE: {TEMPERATE: 1.0, COLD: 1.25, HOT: 0.75}` (cold-leaning - correct bonus for Wool, see [04](04_material_process_chains.md)) |
| `room/PASTURE_AUR.txt`/`_BALTI`/`_GLOBDIEN` | Specialty-good pasture outputs are much higher than their `MEAT` rate: Aur `LEATHER: 0.56` = `MEAT: 0.56` (1x); Globdien `EGG: 1.05` vs `MEAT: 0.35` (3x); Onx `COTTON: 1.8` vs `MEAT: 0.7` (2.6x) - a pasture's "specialty" output tracks roughly 1-3x its meat rate, not a crop-farm's rate |
| `room/REFINER_WEAVER.txt` (original vanilla recipe, since overwritten - see [05](05_integration_and_tech.md)) | `IN: {COTTON: 2} OUT: {FABRIC: 2}`, 1:1; `UPGRADES` 3 tiers (`BOOST: 0 → 1.0 → 2.0`) |
| `room/WORKSHOP_TAILOR.txt` (original vanilla recipes, kept at indices 0-2 after the merge) | `LEATHER:4 → CLOTHES:3.0`, `FABRIC:4.0 → CLOTHES:3.0`, `LEATHER:2.0 → ARMOUR_LEATHER:0.25` - multi-recipe rooms are normal vanilla pattern |
| `stats/equip/civic/_CLOTHES.txt` | `MAX_AMOUNT: 6`, `WEAR_RATE: 0.25`, `DEFAULT_TARGET: 1`, `BOOST: {PHYSICS_RESISTANCE_HOT: 1, PHYSICS_RESISTANCE_COLD: 1.5}`, `STANDING: {CHILD: true, CITIZEN: 7.0, SLAVE: 5.5, NOBLE: 2, EXPONENT: 0.5, PRIO: 10}` |
| `stats/equip/civic/JEWELRY.txt` | `MAX_AMOUNT: 3`, `WEAR_RATE: 0.5`, `DEFAULT_TARGET: 0`, `STANDING: {CITIZEN: 0.5, SLAVE: 0, NOBLE: 3, PRIO: 10}` |
| `tech/AGRI.txt` (Cotton) | `COTT01` (`LEVEL_MAX: 7`, `COSTS.CIVIC_INNOVATION: 4`, `BOOST.ROOM_FARM_COTTON>ADD: 0.15`) → `COTTt1` → `COTT02` |
| `tech/AGRI.txt` (Mushroom, locked) | `MUSH00` (`LEVEL_MAX: 1`, `COSTS: 10`, `UNLOCKS_FACTION: [ROOM_FARM_MUSHROOM]` - no upgrade suffix, gates the base room) |
| `tech/REFINER.txt` (Weaver) | `WEAV01`→`WEAV02` (`+0.15` each) → `WEAVu1` (`COSTS: 58`, unlocks tier 1) → `WEAV03`/`WEAV04` (up to `+0.18`) → `WEAVu2` (`COSTS: 84`, tier 2) → `WEAVc0` (`COSTS.CIVIC_KNOWLEDGE: 35`, efficiency) |
| `race/nobility/FARM4.txt` | Race-trait file granting `BOOST: {ROOM_FARM_COTTON>ADD: 0.25, ROOM_REFINER_WEAVER>ADD: 0.1}` - races can skew toward rooms via the same `BOOST` mechanism |

## Proposed values, per material

Starting points to playtest and retune, each derived by comparison to a
specific real vanilla number above. Room names reflect the consolidated
Spinner/Weaver/Tailor - see [04](04_material_process_chains.md).

### Plant Fibre

| File | Proposed values |
|---|---|
| `resource/COTTON_THREAD.txt` | `DEGRADE_RATE: 0.05`, `CATEGORY_DEFAULT: 1` |
| `room/REFINER_SPINNER.txt` (Cotton recipe) | `IN: {COTTON: 2} OUT: {COTTON_THREAD: 2}` |
| `room/REFINER_WEAVER.txt` (Cotton Thread recipe) | `IN: {COTTON_THREAD: 2} OUT: {FABRIC: 2.2}` |

- Spinner ratio is lossless (`2:2`) - spinning twists fiber, doesn't
  ferment it away like retting.
- `FABRIC: 2.2` (vs. vanilla's original `2.0`, before the Weaver recipe was
  replaced) rewards the extra building; there's no more direct Cotton path
  now that the Spinner step is mandatory for every fiber - see
  [05](05_integration_and_tech.md).
- Spinner `WORK.FULFILLMENT: 0.1` (vs. Weaver's `0.5`) - spinning is
  monotonous cottage-industry labor.

### Wool

| File | Proposed values |
|---|---|
| `resource/WOOL.txt` | `DEGRADE_RATE: 0.05`, `CATEGORY_DEFAULT: 1` |
| `room/PASTURE_ONX_WOOL.txt` | `OUT: {WOOL: {PLAYER: 2.5, AI_RATE: 5.0, AI_RECOVERY: 0.5}, MEAT: {PLAYER: 0.1, AI_RATE: 0.16, AI_RECOVERY: 0.5}, LIVESTOCK: {PLAYER: 0.0875, AI_RATE: 0.875, AI_RECOVERY: 0.1}}` |
| `resource/WOOL_THREAD.txt` | `DEGRADE_RATE: 0.05`, `CATEGORY_DEFAULT: 1` |
| `room/REFINER_SPINNER.txt` (Wool recipe) | `IN: {WOOL: 2} OUT: {WOOL_THREAD: 2}` |
| `resource/WOOL_FABRIC.txt` | `DEGRADE_RATE: 0.05`, `CATEGORY_DEFAULT: 2` |
| `room/REFINER_WEAVER.txt` (Wool Thread recipe) | `IN: {WOOL_THREAD: 2} OUT: {WOOL_FABRIC: 2.2}` |
| `room/WORKSHOP_TAILOR.txt` (Wool Fabric recipe) | `IN: {WOOL_FABRIC: 4.0} OUT: {CLOTHES: {PLAYER: 3.0}}` |

**`WOOL: 2.5` - went through three revisions before landing here, all
worth recording:**

1. First guess was `3.0`, matched to `FARM_COTTON`'s crop-farm rate - wrong
   comparison, a farm harvests continuously across a field, a pasture
   shears an individual animal roughly once a year.
2. Revised to `1.8`, matching Onx's own existing `COTTON` rate exactly, on
   the reasoning that a sheep's fleece grows at a fixed rate per year
   regardless of whether that same animal is also raised for meat -
   dedicating a herd to wool changes what you *stop* getting (routine
   slaughter), not how much wool each individual animal grows. That
   reasoning holds for one animal across different *management* choices,
   but not across differently-*bred* stock.
3. Revised to `3.0` (a `1.67x` multiplier over `1.8`), reasoning that
   `PASTURE_ONX_WOOL` represents a selectively-bred fiber bloodline, and
   real wool breeds (Merino) meaningfully outyield meat breeds. **Anachronistic** -
   Merino-scale breed specialization is a late-medieval/early-modern
   (Spanish, later Australian) phenomenon, centuries past this mod's
   Bronze/Iron Age setting.
4. **Settled on `2.5`** (a `1.35x` multiplier), using the actual Bronze Age
   equivalent instead: the "Secondary Products Revolution" (~4000-3000 BCE)
   selectively bred sheep away from their ancestral shedding coat into a
   continuously-growing fleece - a real wool/non-wool distinction for this
   era, but a far more modest yield gap (~1.3-1.4x, from ration/yield
   estimates) than any modern breed comparison. Cross-checked against the
   game's own full spread: staple goods (`GRAIN: 4`, `COTTON: 3`) sit well
   above rare-good territory (`HERB`/`OPIATES: 0.25`, mine `GEM: 0.2`/
   `SITHILON: 0.1`) - wool was a Bronze Age Mesopotamian staple export on
   par with grain, not a luxury, so it belongs in the staple tier alongside
   Cotton, not down near the rarities. `2.5` sits there while staying a
   meaningful step below Cotton's `3` (sheep husbandry taking more land per
   unit of output than field cropping is a reasonable read of that gap).
   `AI_RATE: 5.0` keeps the same 2x PLAYER:AI_RATE ratio vanilla
   `PASTURE_ONX` uses for `COTTON` (`1.8`:`3.6`).

- `MEAT: 0.1` (vs. vanilla `PASTURE_ONX`'s `0.7`) encodes "wool herd, not a
  food herd" - a trickle from natural attrition. `LIVESTOCK` unchanged.
- `BONUS.CLIMATE` carried from vanilla `PASTURE_ONX` (cold-leaning) - was a
  latent mismatch for Cotton, correct for Wool.
- Tailor ratio matches vanilla's `4:3` exactly - Wool's differentiation is
  climate/output/tech, not the tailoring step.

### Silk

| File | Proposed values |
|---|---|
| `resource/RAW_SILK.txt` | `DEGRADE_RATE: 0.08`, `CATEGORY_DEFAULT: 1` |
| `room/PASTURE_SILKWORM.txt` | Zero-input (Pastures can't have `IN`, see [06](06_misc_notes.md)) - `OUT: {RAW_SILK: {PLAYER: 0.5, AI_RATE: 0.5, AI_RECOVERY: 0.5}}`; `BONUS.CLIMATE: {COLD: 0.1, TEMPERATE: 1.0, HOT: 1.2}` |
| `resource/SILK.txt` | `DEGRADE_RATE: 0.03`, `CATEGORY_DEFAULT: 2` |
| `room/REFINER_WEAVER.txt` (Raw Silk recipe) | `IN: {RAW_SILK: 2} OUT: {SILK: 2}` |
| `resource/FINE_CLOTHES.txt` | `DEGRADE_RATE: 0.1`, `CATEGORY_DEFAULT: 3` |
| `room/WORKSHOP_TAILOR.txt` (Silk recipe) | `IN: {SILK: 4} OUT: {FINE_CLOTHES: {PLAYER: 1.5}}` |
| `stats/equip/civic/_FINE_CLOTHES.txt` | `RESOURCE: FINE_CLOTHES`, `MAX_AMOUNT: 4`, `WEAR_RATE: 0.15`, `DEFAULT_TARGET: 0`, `STANDING: {CITIZEN: 0.25, SLAVE: 0, NOBLE: 4.0, PRIO: 10}` |

- `PASTURE_SILKWORM`'s `RAW_SILK: 0.5` is deliberately low - lower than
  Wool's `1.8` and well below any farm's crop rate - since removing the
  Mulberry input also removed the old `3:1` conversion-loss scarcity lever;
  scarcity now has to come entirely from this one number plus
  `WORKSHOP_TAILOR`'s halved `FINE_CLOTHES` output. Not yet
  calibrated against real play - watch this one closely.
- `SILK`'s `DEGRADE_RATE: 0.03` is lower than plant fiber's `0.05` - silk
  thread keeps better.
- Gated behind research (`SILK00` tech) - see [05](05_integration_and_tech.md).

### Dye

| File | Proposed values |
|---|---|
| `resource/DYE_PLANT.txt` + `growable/DYE_PLANT.txt` | `DEGRADE_RATE: 0.05`; `GROWTH_VALUE: 0.3`; `CLIMATE_BONUS: {COLD: 0.5, TEMPERATE: 1.0, HOT: 0.8}` |
| `room/FARM_DYE.txt` | `OUT.DYE_PLANT: {PLAYER: 2.5, AI_RATE: 2.4}` |
| `resource/DYE.txt` | `DEGRADE_RATE: 0.05`, `CATEGORY_DEFAULT: 1` |
| `room/REFINER_DYER.txt` | `IN: {DYE_PLANT: 3} OUT: {DYE: 1}` |
| Dyed recipes (all in `WORKSHOP_TAILOR`) | `WOOL_FABRIC:4.0 + DYE:1.0 → DYED_CLOTHES:{PLAYER:3.0}`; `SILK:4 + DYE:1.0 → DYED_FINE_CLOTHES:{PLAYER:1.5}`; `FABRIC:4.0 + DYE:1.0 → DYED_CLOTHES:{PLAYER:3.0}` |
| `resource/DYED_CLOTHES.txt` / `DYED_FINE_CLOTHES.txt` | `DEGRADE_RATE: 0.1`, `CATEGORY_DEFAULT: 3` |
| `stats/equip/civic/DYED_CLOTHES.txt` | `MAX_AMOUNT: 6`, `WEAR_RATE: 0.25`, `DEFAULT_TARGET: 0`, `STANDING: {CITIZEN: 2.0, SLAVE: 0.2, NOBLE: 2.5, PRIO: 9}` |

- Dye's `3:1` ratio is lossy like Silkworm Breeding's, but `FARM_DYE`'s
  output (`2.5`) is much higher than `FARM_MULBERRY`'s (`1.5`) - dye is a
  broadly accessible status good, not gated as hard as silk.
- `DYE:1.0` cost is added on top of each recipe with output unchanged -
  dye is a cost/status premium, not a production bottleneck.
- `DYED_CLOTHES`'s `STANDING` is flat (`CITIZEN: 2.0, NOBLE: 2.5`) vs.
  `_FINE_CLOTHES`'s steep skew (`CITIZEN: 0.25, NOBLE: 4.0`) - dye is a
  broad nice-to-have, silk is a noble-exclusive good.

## Tech-tree values

See [05_integration_and_tech.md](05_integration_and_tech.md) for the full
node lists and grid placement. Costs follow the vanilla `COTT0x`/`WEAV0x`
shape (`CIVIC_INNOVATION: 4-12` per output-boost tier, `CIVIC_KNOWLEDGE: 20`
for efficiency capstones, `CIVIC_INNOVATION: 10` for the Silkworm Breeder's
unlock node, matching `MUSH00`'s own `10`).

## Differentiation summary

| Material | Growth/pasture climate | Output rate | Scarcity lever | Class skew |
|---|---|---|---|---|
| **Cotton/Plant Fibre** (vanilla baseline) | Hot-leaning | Standard | None | None |
| **Wool** | Cold-leaning | ~1.35x Onx's own fiber rate (Bronze Age wool-breeding specialization) | None | Slight, via `CLOTHES`' `PHYSICS_RESISTANCE_COLD` |
| **Silk** | N/A (zero-input pasture) | Low (`0.5` base rate, halved again at Tailor) | Low base rate + research-gated | Strong - `_FINE_CLOTHES`, `NOBLE: 4.0` |
| **Dye** | Broad | Moderate | `REFINER_DYER`'s `3:1` ratio, high base output | Mild - `DYED_CLOTHES`, flat `CITIZEN: 2.0`/`NOBLE: 2.5` |
