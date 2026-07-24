# Textiles Mod — Design Docs

Working documentation for the textile-economy overhaul (wool, flax, silk, hemp
alongside cotton, later dyes), inspired by *The Fabric of Civilization*.

* [01. Recipe Walkthrough](01_recipe_walkthrough.md) — the generalized, repeatable
  steps used to build the flax chain, ready to copy for hemp/wool/silk.
* [02. Custom Art Guide](02_custom_art_guide.md) — what art assets exist per
  material/room, real pixel dimensions from the vanilla files, how to swap in
  your own.
* [03. Economic Levers](03_economic_levers.md) — every config key that changes
  cost/price/labor/growth-time/tech/happiness, with concrete recommendations
  for differentiating each fiber.
* [04. Material Process Chains](04_material_process_chains.md) — the full
  building list for cotton (reference), flax, hemp, wool, silk, including how
  to have multiple building tiers for one stage.
* [05. Integration & Tech Tree](05_integration_and_tech.md) — how to fold this
  into the Tailor, the citizen equipment/happiness system, and the research
  tree, plus an honest note on what tech *can't* gate.
* [06. Misc Notes](06_misc_notes.md) — loose ends, risks, and ideas worth
  knowing about before going further.

Status as of this writing: flax v1 (`FARM_FLAX` → `REFINER_RETTER_FLAX` →
`LINEN`) is built, installed, and confirmed working in-game with placeholder
art borrowed from Cotton/Fabric/Weaver. Vanilla `COTTON`'s display name was
also fixed from the generic "Fibre" to "Cotton" via a partial text-file
override. Nothing for hemp/wool/silk/dyes exists yet — these docs are the plan
for building them.
