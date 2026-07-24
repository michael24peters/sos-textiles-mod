# Economic Levers

Every config key that changes cost, price, labor, growth time, research, or
happiness/class impact — with real key names, not vague suggestions — plus a
concrete recommendation for how to use them to make wool/flax/silk/hemp feel
different from each other rather than reskinned cotton.

## Per-resource levers (`assets/init/resource/<X>.txt`)

| Key | Effect |
|---|---|
| `DEGRADE_RATE` | How fast stockpiled units spoil (0.0–1.0). Higher = more storage-loss pressure, pushes toward just-in-time production. |
| `PRICE_CAP` | Multiplier cap on the *buy* price (0.0–1.0 of some base). |
| `PRICE_MUL` | Multiplier on the *sell* price (0.0–100.0). This is your margin lever between raw fiber and finished cloth. |
| `CATEGORY_DEFAULT` | Just UI sort order — cosmetic, not economic. |

## Per-growable levers (`assets/init/resource/growable/<X>.txt`)

| Key | Effect |
|---|---|
| `GROWTH_VALUE` | Growth speed. This is your "growth time" lever. |
| `SEASONAL_OFFSET` | Shifts the growing season. |
| `CLIMATE_BONUS.{COLD,TEMPERATE,HOT}` | Where the plant thrives. This is a strong differentiation lever — see recommendations below. |

## Per-room levers (any `FARM_`/`PASTURE_`/`REFINER_`/`WORKSHOP_`)

| Key | Effect |
|---|---|
| `RESOURCES` + `AREA_COSTS` | Build materials and per-tile cost — your start-up cost lever. |
| `ITEMS[].COSTS` | Per-furniture-piece build cost (multiplies `RESOURCES`). |
| `BONUS.CLIMATE` | Production multiplier by climate — same lever as growable `CLIMATE_BONUS`, but at the *building* level (relevant for `PASTURE_`/`REFINER_` where there's no growable resource). |
| `WORK.SHIFT_OFFSET` / `FULFILLMENT` / `ACCIDENTS_PER_YEAR` / `NIGHT_SHIFT` / `HEALTH_FACTOR` | Labor conditions: worker morale while working here, injury risk, whether it runs at night. |
| `INDUSTRY(.IES).OUT.<RES>.PLAYER` / `.AI_RATE` / `.AI_RECOVERY` | Production rate — separately tunable for the player vs. AI factions. |
| `INDUSTRIES.INDUSTRY.IN`/`OUT` ratio | The conversion efficiency of a recipe. E.g. cotton's weaver is 1:1 (`COTTON: 2 → FABRIC: 2`); making a recipe lossy (e.g. `FLAX: 3 → LINEN: 2`) is a realistic way to model retting/scutching material loss and make one material cheaper-but-less-efficient than another. |
| `UPGRADES[].RESOURCE_MASK` / `.BOOST` / `.AI` | Progressive upgrade tiers for the *same* building — the vanilla-idiomatic way to add "a better version" without a whole new room. |
| `STORAGE` | (Refiner/Workshop only) buffer capacity — affects how bursty vs. steady the output feed is. |

## Tech-tree levers (`assets/init/tech/<CATEGORY>.txt`)

| Key | Effect |
|---|---|
| `COSTS.CIVIC_INNOVATION` / `CIVIC_KNOWLEDGE` | Research cost — your "how much research is required" lever, directly. |
| `LEVEL_MAX` | How many times a tech node can be leveled up (each level typically reapplies its `BOOST`). |
| `REQUIRES_TECH_LEVEL` | Prerequisite chain — lets you gate an efficiency tech behind an earlier one. |
| `BOOST.ROOM_<ROOMKEY>>ADD` | Output-rate research (e.g. `ROOM_REFINER_WEAVER>ADD: 0.15`). These booster keys are auto-generated for *any* room you add — `ROOM_REFINER_RETTER_FLAX` and `ROOM_FARM_FLAX` already exist as valid boost targets with zero extra registration. |
| `BOOST.ROOM_CONSUMPTION_<ROOMKEY>_<i>>ADD` | Efficiency research (lower input use per recipe `i`) — same auto-generation. |
| `BOOST.EQUIP_LEVEL_TOOL_<ROOMKEY>>ADD` | Unlocks/raises the tool tier workers can use in that room. |
| `UNLOCKS_FACTION: [ROOM_<ROOMKEY>_UPGRADE_N]` | Unlocks a room's `UPGRADES[]` tier N — this is how vanilla tech actually gates *access* to better versions of a building, rather than gating the base building itself (see [05](05_integration_and_tech.md) for why that distinction matters). |

## Class-interplay levers (`assets/init/stats/equip/civic/<X>.txt`)

This is the real lever for "how do peasants vs. nobles vs. kings relate to
this differently" — and it's a completely separate system from the
resource/room economy above. Any citizen-worn good (clothes, jewelry, and
whatever new ones you add) is defined here, independent of which room
produced it:

| Key | Effect |
|---|---|
| `RESOURCE` | Which resource this equip slot consumes to "wear." |
| `MAX_AMOUNT` | Equip-level cap (raisable via tech, same as room `UPGRADES`). |
| `WEAR_RATE` | How fast equipped items wear out — your ongoing-consumption-velocity lever. |
| `DEFAULT_TARGET` | Starting demand level. |
| `BOOST.<KEY>` | Side-effects of wearing it — vanilla `CLOTHES` grants `PHYSICS_RESISTANCE_HOT`/`_COLD`. |
| `STANDING.{CHILD,CITIZEN,SLAVE,NOBLE}` | **How much each class cares about this good for happiness/standing.** Vanilla `_CLOTHES.txt`: `CITIZEN: 7.0, SLAVE: 5.5, NOBLE: 2, CHILD: true`. Vanilla `JEWELRY.txt`: `NOBLE: 3, CITIZEN: 0.5, SLAVE: 0` — jewelry is already a noble-skewed good in vanilla. This is your template. |

Multiple equip files can exist side by side (`_CLOTHES` and `JEWELRY` both
already do) — nothing stops you from adding a wholly new equip slot, e.g. a
`_FINE_CLOTHES` fed by silk, with its own independent `STANDING` weighting.
This is the mechanism recommended in [05](05_integration_and_tech.md) for
making silk genuinely matter to nobility in a way cotton clothes never will.

## Recommended differentiation per material

| Material | Growth/pasture climate | Output rate | Tech cost | Class skew |
|---|---|---|---|---|
| **Cotton** (vanilla, reference) | Hot-leaning | Standard | Standard | None — everyday good |
| **Flax/Linen** | Temperate/cool-leaning | Standard | Standard | None — everyday good, maybe cooler-climate alternative to cotton |
| **Hemp** | Broad tolerance, fast growth | High (bulk fiber) | Cheap/early | None — utilitarian good; consider feeding a second, non-clothing resource (rope/cordage) per [04](04_material_process_chains.md) and [06](06_misc_notes.md) |
| **Wool** | Cold-leaning (pasture `BONUS.CLIMATE`) | Standard, tied to flock size | Standard | Slight warmth premium — lean `BOOST.PHYSICS_RESISTANCE_COLD` harder than cotton/linen's clothes |
| **Silk** | Narrow (mulberry needs warm/temperate) | Low (luxury scarcity) | Expensive, late-tier | Strong — feed a *new* `_FINE_CLOTHES`-style equip slot weighted hard toward `NOBLE` |

The scarcity/luxury feel for silk should come from **output rate** and
**tech cost**, not from an artificial hard lock — nothing in this system
prevents a citizen from wearing silk if it's available and cheap, same as
nothing stops a peasant from buying jewelry today. Scarcity has to be
*produced* economically (low output, high build/tech cost) for the class
distinction to actually bite.
