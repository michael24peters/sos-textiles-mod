# Textiles Mod

This mod adds historically-inspired textile systems. Some of the current game
systems prevent creating the real-world systems with exactness; likewise,
a certain level of abstraction is applied with consistency throughout the game
with which I didn't want to do away.

Much of this textile work was inspired by my readings of *The Fabric of
Civilization* by Virginia Postrel. I enjoy the simulation aspect of Songs of
Syx, and having spent some time reading about the ancient world, I felt like
some important aspects of ancient society were not included, or at least
abstracted too much for my taste. So I thought I would add in some systems
that I think slot in well with the existing game.

While real-world textiles and dyes were varied, I tried to simplify where I
felt there was not enough meaningful difference to justify inclusion. For
example, I simplified the numerous Plant Fibers -- Hemp, Flax, Cotton -- down
to just the vanilla Cotton. Likewise, I simplified the numerous Dyes -- Woad,
Indigo, Madder, Murex -- down to just one Dye resource.

Features include:

- Three textile resources: Cotton, Wool, and Silk
- Three intermediate resources: Cotton Thread, Wool Thread, and Silk Thread. 
- One new resource: Dye
- One new animal: Silkcrawler
- Two new buildings: Spinner (Refiner), Dyer (Refiner), Silkcrawler Breeder (Husbandry)
- New crafted resources: Wool Clothes, Fine Clothes, Dyed Clothes, Dyed Fine Clothes
- New recipes for all these additions in Weaver and Tailor.

Let me know if you run into any bugs, issues, or in-game inconsistencies. This
is my first time making a mod (and I'm not even that experienced in Songs of
Syx).

## Process

I will detail each resource process below for clarity. In parentheses are the
resources between each stage/structure.

- Cotton Farm → (Cotton) → Spinner → (Thread) → Weaver → (Fabric) → Tailor → (Clothes)
- Onx Pasture → (Wool) → Spinner → (Wool Thread) → Weaver → (Wool Fabric) → Tailor → (Wool Clothes)
- Silkcrawler Breeder → (Raw Silk) → Weaver → (Silk) → Tailor → (Fine Clothes)
- Dye Farm → (Dye Plant) → Dyer → (Dye) 

## Designer's Notes

There are a lot of untested, open questions awaiting feedback. Here's a short
list:

- Wool and Cotton Clothes both produce Dyed Clothes when processed by Dyer.
- Naming conventions: 
  - Cotton Thread → Fabric, but Wool Thread → _Wool_ Fabric
  - Cotton Fabric and Wool Fabric → Clothes
  - Cotton _Fabric_ → Clothes, but Silk → _Fine_ Clothes
- The Silkcrawler produces no meat. Seems appropriate from historical context of use case.
  - Currently, hunting a silkcrawler gives you silk.
- Game constraints prevent Mulberry Farm → Silk_worm_ breeder (with Mulberry requirements).
- Silk does not require a Spinner; silk was not historically spun once collected.
- First time doing assets like this, so they might not be at the same standard as the vanilla assets.
- I tried to minimize new buildings and use new recipes where possible.

## AI Disclosure

AI was used for initial research of the source code and finding the relevant
files that needed to be made, modified, and updated. I tried using AI for
documentation and such, but it ended up being kind of lengthy for little gain,
so I stripped out what I could from the config files and wrote the rest myself
(like this doc you're currently reading, which is written by me). I also
periodically had AI check my work, though actual edits were done by hand.

All art was drawn in GIMP by me and me alone, using the in-game existing assets
as a starting point where relevant.
