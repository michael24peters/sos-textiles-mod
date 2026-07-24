# Integration & Tech Tree

## Extending the Tailor without touching the vanilla file

Vanilla `WORKSHOP_TAILOR.txt` has three recipes: `LEATHER → CLOTHES`,
`FABRIC → CLOTHES`, `LEATHER → ARMOUR_LEATHER`. The tempting move is to add
`LINEN → CLOTHES` etc. as a fourth recipe in that same file — but editing a
vanilla file means either a full `__OVERWRITE: true` replacement (you now own
keeping every future vanilla balance change in sync by hand) or the newer
partial-key override (only proven in the docs for simple scalar keys, not for
appending a new entry into an `INDUSTRIES` list — untested for this case).

**Recommended instead: add a brand-new, differently-named `WORKSHOP_` room**
that also produces `CLOTHES`, e.g. `WORKSHOP_TAILOR_LINEN.txt`, with its own
single recipe `LINEN → CLOTHES`. This is purely additive (per
`doc/README.md`'s "Adding your own custom config files" pattern, the same
mechanism that let `FARM_FLAX`/`REFINER_RETTER_FLAX` just work with zero
registration) and both rooms feed the exact same `CLOTHES` resource and the
same `_CLOTHES.txt` civic equip need — citizens don't care which building made
their clothes. Repeat per material: `WORKSHOP_TAILOR_HEMP`,
`WORKSHOP_TAILOR_WOOL`. This is "seamless" in the sense that matters most: the
player's citizens automatically consume whichever clothing-producing
resource is available, no new need/UI concept required.

For silk specifically, don't feed plain `CLOTHES` — see below.

## A genuinely new consumption tier for silk (and later, dyed goods)

This is the actual lever for making the new systems feel integrated rather
than bolted-on: a **new civic equip slot**, parallel to how `_CLOTHES.txt` and
`JEWELRY.txt` already coexist as independent equip categories.

`assets/init/stats/equip/civic/_FINE_CLOTHES.txt`:
```
RESOURCE: SILK,
MAX_AMOUNT: 4,
WEAR_RATE: 0.15,
DEFAULT_TARGET: 0,
BOOST: {
	<something status-flavored, or leave empty — real luxury goods
	 don't need a utility payoff, the STANDING weighting is the point>
},
STANDING: {
	CITIZEN: 0.25,
	SLAVE: 0,
	NOBLE: 4.0,
	PRIO: 10,
},
```
This automatically creates a `EQUIP_CIVIC_FINE_CLOTHES` race-stat key (per
`doc/config/race.md`'s dynamic-EQUIP-stat rule: leading `_` collapses, so
`_FINE_CLOTHES.txt` → `EQUIP_CIVIC_FINE_CLOTHES`), which you can then also
use in race files' `STANDING`-adjacent tuning the same way any other stat is
used. Nobility now has a real, mechanically distinct reason to want silk that
peasants structurally don't — not a flavor label on an identical good.

## Folding into the research tree

Every room you add automatically gets its own boostable keys with zero
registration — `ROOM_FARM_FLAX`, `ROOM_REFINER_RETTER_FLAX`,
`ROOM_CONSUMPTION_REFINER_RETTER_FLAX_0`, `EQUIP_LEVEL_TOOL_FARM_FLAX`, etc. all
already exist as valid boost targets the moment the room config exists (this
is how `doc/config/boosters.md`'s dynamic keys work — same mechanism that
makes `MONUMENTS_MONUMENT_*` and `SERVICE_*` race-stats appear automatically
for any room you add).

So a tech node that boosts your Retting Shed is just:
```
FLAX01: {
	LEVEL_MAX: 10,
	COSTS: { CIVIC_INNOVATION: 4, },
	BOOST: { ROOM_REFINER_RETTER_FLAX>ADD: 0.15, },
},
```
The **placement** of that node is the open question. Vanilla's tech files are
one file per category tab (`AGRI.txt`, `REFINER.txt`, `WORKSHOP.txt`, ...),
each with a fixed `CATEGORY` id and a `TREE` grid that's a fixed-width layout
(15 columns observed in both `AGRI.txt` and `WORKSHOP.txt`) — this strongly
suggests category tabs are a fixed, hardcoded set of UI tabs, not something
you can add a wholly new tab for via config alone. **Untested — treat as a
real constraint until proven otherwise**, and default to adding new tech nodes
as new columns inside the existing `AGRI.txt` (farm techs) and `REFINER.txt`
(refiner techs) category files rather than trying to invent a "Textiles" tab.

Whether you can *add* a new tree column/new `TECHS` entries into those vanilla
files via the partial-override mechanism, or whether you need a full
`__OVERWRITE: true` replacement (and therefore own resyncing the entire
vanilla tech file by hand across game updates), is genuinely unverified from
the docs — the only demonstrated partial-override example
(`doc/README.md`'s `CANTOR.txt`/`PLAYABLE: true`) is a single top-level
scalar, not adding new keys into a nested `TREE`/`TECHS` structure. **Test this
in isolation first**: try adding one new `TREE` column entry + one new
`TECHS:{}` node to a copy of `AGRI.txt` without `__OVERWRITE`, confirm in-game
whether it merges or the file is ignored/errors, before building out a full
tech tree on that assumption.

## What tech can and can't gate (read before designing "must research X to grow flax")

A `grep` across every vanilla `room/*.txt` shows **zero** uses of the
documented `REQUIRES` key, even though `doc/config/room.md`'s General section
lists `REQUIRES.<COMPARATOR>` as available on any room. `REQUIRES` is real and
used elsewhere (world-building files, tech files themselves, player titles,
events) — just never to gate a settlement room's buildability. Nor is there a
`REQUIRES` comparator value in `doc/res/comparator_values_all.md` for
"has researched tech X to level Y" — the tech tree's own
`REQUIRES_TECH_LEVEL` only gates *other tech nodes*, and `UNLOCKS_FACTION`
only gates *upgrade tiers within a room you can already build*.

Net effect: vanilla's model is **"all base industries are available from
turn one; research makes them better,"** not a 4X-style unlock tree. If your
goal is "you can't build a Flax Farm until you've researched Flax
Farming" — that's a real design deviation from how this game's economy tech
works everywhere else, and would need either (a) an untested `REQUIRES` on a
room (worth a small isolated test), or (b) Java code. If instead your goal is
closer to "flax farming exists from the start but only produces well /
unlocks its upgrade tier after research" — that's exactly the existing
pattern, zero risk, and probably reads just as well to a player as true
gating would.
