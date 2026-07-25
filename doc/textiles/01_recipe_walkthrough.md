# Recipe Walkthrough: Adding a New Fiber Chain

Generalized steps for adding a resource → room → room chain, based on the
Wool/Silk/Dye chains actually built. Config-only, no Java code.

## Step 0: Room filename prefix — read this first

Room type is determined by the filename prefix, matched against a fixed
list in the game's code (`FARM_`, `REFINER_`, `WORKSHOP_`, `PASTURE_`,
`MINE_`, etc. — see [doc/howto/make_custom_room.md](../howto/make_custom_room.md)
for the full list). If the prefix isn't recognized, the room is silently
never constructed, and any resource it was meant to produce ends up with
zero producers — crashes the game the moment a settlement is created. See
[06_misc_notes.md](06_misc_notes.md) for what this looks like when it goes
wrong. **Only use a listed prefix.**

## Step 1: Find the closest vanilla template

Vanilla files live in the game install's `base/data.zip` (not
`SongsOfSyx.jar`, which only has compiled code). Unzip and browse
`data/assets/init/...` and `data/assets/text/...`. Find the closest
existing resource/room pair and copy its shape — this inherits real,
balanced numbers instead of guessing furniture costs from scratch.

## Step 2: Decide the chain shape

Pick which room types the chain needs and how many links:

```
FARM_<MATERIAL> or PASTURE_<ANIMAL>  →  REFINER_<NAME>  →  WORKSHOP_<NAME>
```

Not every chain needs all three stages — see
[04_material_process_chains.md](04_material_process_chains.md) for the
actual shapes used (Plant Fibre, Wool, Silk, Dye).

## Step 3: Add the raw resource

- `assets/init/resource/<MATERIAL>.txt` — icon, color, degrade rate,
  category. Copy the closest vanilla resource file.
- If farmable: `assets/init/resource/growable/<MATERIAL>.txt` — growth
  speed, `CLIMATE_BONUS`.
- `assets/text/resource/<MATERIAL>.txt` — `NAME`/`NAMES`/`DESC`. Use a
  specific name, not a generic placeholder.

## Step 4: Add the producer room

`assets/init/room/FARM_<MATERIAL>.txt` or `PASTURE_<ANIMAL>.txt` —
`GROWABLE`/`ANIMAL`, `RESOURCES`/`AREA_COSTS`, `INDUSTRY.OUT.<MATERIAL>`,
`BONUS.CLIMATE`. Copy the closest vanilla farm/pasture room.

Matching `assets/text/room/FARM_<MATERIAL>.txt` needs `INFO`/`BONUS`/`WORK`/
`STATS` — `STATS` count must exactly match the vanilla farm's stat slots (4
entries: Soil, Farmers, blank, Yearly Output), these are hardcoded per room
type.

## Step 5: Add the refining/crafting room

`assets/init/room/REFINER_<NAME>.txt` or `WORKSHOP_<NAME>.txt` — copy a
vanilla room of the same type wholesale, only change
`INDUSTRIES.INDUSTRY.IN`/`OUT`.

Matching `assets/text/room/REFINER_<NAME>.txt` — `INFO`/`BONUS`/`WORK`
text, `ITEMS` names matching the furniture slot count in the init file.

## Step 6: Build and install

```bash
chmod +x mvnw   # once
./mvnw clean install   # always "clean install", and "./mvnw" not "bash mvnw"
```

Gotchas:

1. **Two candidate mod directories.** Flatpak Steam reads
   `~/.var/app/com.valvesoftware.Steam/.local/share/songsofsyx/mods/`. A
   decoy `~/.local/share/songsofsyx` also exists but isn't read by the
   running game.
2. **Renaming the mod (`pom.xml`'s `mod.name`) needs `mvn clean` first**, or
   you get two mod folders, the stale one still active in-game.
3. **Plain `./mvnw install` does not remove deleted files.** If you delete
   a mod file from source, `install` alone leaves the stale copy in the
   installed mods directory, still active in-game. Always use
   `./mvnw clean install` after deleting or renaming a mod file.

## Step 7: Verify in-game

1. Build menu → correct category → new room appears with name/icon.
2. Place the producer, staff it, confirm it produces the raw resource.
3. Place the refiner/workshop, staff it, confirm input is consumed and
   output accumulates.
4. Trade/market screen — output resource should already be listed and
   tradable, zero extra steps (see [06](06_misc_notes.md)).
