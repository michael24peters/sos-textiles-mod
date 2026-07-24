# Recipe Walkthrough: Adding a New Fiber Chain

This is exactly what was done to add Flax, generalized so it can be repeated
for Hemp, Wool, and Silk. None of this requires Java code — it's all
config files mirroring the vanilla `data/assets/...` folder structure.

## Step 0: Find the closest vanilla template

Before writing anything, find the vanilla files that already do roughly what
you want, and read them fully. They live inside the game install's
`base/data.zip` (NOT inside `SongsOfSyx.jar` — that only has compiled code):

```
~/.var/app/com.valvesoftware.Steam/.local/share/Steam/steamapps/common/Songs of Syx/base/data.zip
```

Unzip it somewhere and browse `data/assets/init/...` and `data/assets/text/...`.
For flax, the template was cotton's chain:

* `assets/init/resource/growable/COTTON.txt` + `assets/init/resource/COTTON.txt`
* `assets/init/room/FARM_COTTON.txt`
* `assets/init/room/REFINER_WEAVER.txt`
* matching `assets/text/resource/*.txt` and `assets/text/room/*.txt`

For hemp, flax's own new files are now the template (they're nearly identical
bast fibers). For wool, `PASTURE_AUR.txt` is the template. See
[04_material_process_chains.md](04_material_process_chains.md) for which
vanilla file to start from for each material.

## Step 1: Decide the chain shape

Pick which vanilla room *types* you're reusing and how many links the chain
has. For flax v1 we deliberately used the shortest possible chain to prove the
pipeline end-to-end:

```
FARM_<MATERIAL>  (growable room)  →  REFINER_<NAME>  (fiber → cloth)
```

Cotton's own chain is one link longer (it also has `WORKSHOP_TAILOR` turning
`FABRIC` into `CLOTHES`). Decide up front whether you're stopping at a
tradable intermediate good (like flax v1 did) or going all the way to a
consumer good — either is fine as a milestone, just know which one you're
building toward before writing files.

## Step 2: Add the raw resource

Two (or three, for growables) files:

`assets/init/resource/<MATERIAL>.txt` — the resource's physical properties
(icon, color, degrade rate, category). Copy the closest vanilla resource file
and adjust `COLOR`/`CATEGORY_DEFAULT`/`DEGRADE_RATE`. Point `SPRITE`/`ICON` at
an existing vanilla sprite name as a placeholder (see
[02_custom_art_guide.md](02_custom_art_guide.md)).

If it's a farmable plant, also add
`assets/init/resource/growable/<MATERIAL>.txt` — growth speed, climate
preference, and the field sprite reference. Copy the closest growable
(`COTTON.txt`, `GRAIN.txt`, etc.) and adjust `CLIMATE_BONUS` to fit the real
plant's preferences (flax likes temperate/cool climates, unlike cotton's
heat preference).

Add `assets/text/resource/<MATERIAL>.txt` with `NAME`/`NAMES`/`DESC` — this is
what the player actually sees. Do **not** reuse a vanishingly generic name
like vanilla's own "Fibre" for cotton — see the naming fix in
[06_misc_notes.md](06_misc_notes.md).

## Step 3: Add the producer room

`assets/init/room/FARM_<MATERIAL>.txt` (or `PASTURE_<ANIMAL>.txt` for an
animal product) — `GROWABLE: <MATERIAL>`, `RESOURCES`/`AREA_COSTS` for build
cost, `INDUSTRY.OUT.<MATERIAL>` for output rate, `BONUS.CLIMATE` for climate
multiplier. Copy the closest vanilla farm/pasture room.

Add the matching `assets/text/room/FARM_<MATERIAL>.txt` with `INFO`, `BONUS`,
`WORK`, and `STATS` text — `STATS` count must exactly match the vanilla farm's
stat slots (4 entries: Soil, Farmers, blank, Yearly Output) since these are
hardcoded per room type, not something you can add/remove.

## Step 4: Add the refining room

`assets/init/room/REFINER_<NAME>.txt` — copy a `REFINER_` room wholesale
(furniture costs, upgrades, sprites, work/noise settings) and only change the
`INDUSTRIES.INDUSTRY.IN`/`OUT` resource keys. This is the fastest way to get a
working, well-balanced room, because you inherit real numbers instead of
guessing furniture costs from scratch.

Add the matching `assets/text/room/REFINER_<NAME>.txt` — `INFO`/`BONUS`/`WORK`
text, and `ITEMS` names matching the number of furniture slots in the init
file (3, if you copied `REFINER_WEAVER.txt`'s shape).

## Step 5: Build and install

```bash
chmod +x mvnw   # once
./mvnw install  # NOT "bash mvnw" — its internal ${0%/*} path logic needs the leading ./
```

Two gotchas found the hard way:

1. **Two candidate mod directories exist on this machine.** The one the
   Flatpak Steam game actually reads is
   `~/.var/app/com.valvesoftware.Steam/.local/share/songsofsyx/mods/`. There's
   a decoy `~/.local/share/songsofsyx` that looks equally plausible but isn't
   read by the running game. If a change doesn't seem to show up, check
   you're looking in the right one.
2. **`mod.install.directory` depends on `mod.name`.** If you ever rename the
   mod in `pom.xml`, run `mvn clean` *before* the rename (or manually delete
   the old-named folder from the mods directory) — otherwise you get two mod
   folders, the stale one still active in-game.

## Step 6: Verify in-game

Launch the game (`mvnw` doesn't do this — launch normally through Steam or
the `.run/*.xml` configs), start/load a save with the mod enabled, and check:

1. Build menu → the right category (Agriculture for a `FARM_`, Industry for a
   `REFINER_`) → your new room appears with its name and icon.
2. Place the producer room on suitable terrain, staff it, let it produce the
   raw resource.
3. Place the refiner, staff it, confirm the raw resource is consumed and the
   output resource accumulates in storage.
4. Check the trade/market screen — the output resource should already be
   listed and tradable with zero extra steps (see
   [06_misc_notes.md](06_misc_notes.md) for why).

## Repeating for the next material

Hemp is close to a pure search-and-replace of the flax files (`FLAX`→`HEMP`,
`LINEN`→ whatever you call hemp cloth, "Flax"→"Hemp", "Retting Shed"→"Hemp
Retting Shed"). Wool and silk need one extra design decision each before you
start copying files — see
[04_material_process_chains.md](04_material_process_chains.md).
