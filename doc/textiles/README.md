# Textiles Mod — Design Docs

Documentation for the textile-economy overhaul: **Plant Fibre**
(cotton/flax/hemp, one abstracted bucket), **Wool**, **Silk**, and a **Dye**
status layer across all three.

* [01. Recipe Walkthrough](01_recipe_walkthrough.md) — steps to add a new
  resource/room chain, including the room-filename-prefix rule.
* [02. Custom Art Guide](02_custom_art_guide.md) — art assets per
  material/room, real pixel dimensions, how to swap in your own.
* [03. Economic Levers](03_economic_levers.md) — every relevant config key,
  verified vanilla values, and the proposed values for every new
  resource/room.
* [04. Material Process Chains](04_material_process_chains.md) — the
  building list for Plant Fibre, Wool, Silk, and Dye.
* [05. Integration & Tech Tree](05_integration_and_tech.md) — Tailor and
  citizen equipment/happiness integration; full tech tree node lists.
* [06. Misc Notes](06_misc_notes.md) — engine gotchas, resolved questions,
  what's still unverified.
* [07. Implementation Log](07_implementation_log.md) — proposed vs.
  as-implemented values, kept current as playtesting retunes numbers.

**Status**: Plant Fibre, Wool, Silk, and Dye are built and installed.
Processing is consolidated into one Spinner (new room), and the Weaver and
Tailor recipes are merged directly into vanilla's own `REFINER_WEAVER`/
`WORKSHOP_TAILOR` (no more duplicate rooms in the build menu) — see
[05](05_integration_and_tech.md). Silk's Raw Silk step is a
Husbandry-category Silkworm Breeder (not a Refiner), zero-input like every
vanilla pasture, gated behind research. A room-prefix bug that crashed
settlement creation was found and fixed — see
[06](06_misc_notes.md#room-filenames-must-start-with-a-recognized-type-prefix).
Tech tree lines are implemented for Spinner/Dyer (Refining, full 6-tier
shape matching every other Refining line), Silkworm Breeder (Husbandry,
locked, full 7-node shape matching Globdien/Warbeast), Dye Farm
(Agriculture), and Dyeing (Crafting) — see [05](05_integration_and_tech.md).
Placeholder art throughout. The earlier Flax proof-of-concept has been
retired. **None of this session's tech tree/consolidation/recategorization
work has been playtested yet** — see [07](07_implementation_log.md) for
what still needs a human to check.
