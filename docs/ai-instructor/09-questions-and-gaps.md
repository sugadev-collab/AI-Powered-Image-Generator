# 09 — Open Questions and Easy-to-Miss Things

## Questions for you (decisions needed)

1. **What will the Image Generator (Scope 2) be?** This is the biggest open decision, because it affects the details of the instruction. Options:
   - **a. Procedural drawing**: C++ code draws each part from shapes, colors and textures. Fast, free, consistent look.
   - **b. Part assembly**: an artist (or you) draws parts, such as trunks and leaf shapes. The generator layers and recolors them. Fast, free, the most consistent look.
   - **c. AI image model** (for example Stable Diffusion). The most variety, but heavy, needs a GPU, and makes a consistent look hard.
   - *Suggestion:* a or b (or both) fit your goals of "same look", "fast" and "0 USD" best. The instruction format in `03` works for all three.
2. **Art style**: pixel art? What are the usual sprite sizes (16×16, 32×32, 64×64)?
3. **Which game engine or language does the game use?** Will the AI Instructor be a C++ library linked into the game, or a separate program the game talks to?
4. **Platforms**: PC only, or also mobile or web?
5. **Internet**: are you OK with the separate Research Tool (see `01-how-it-works.md`) instead of searching during a game request?
6. **Rarity**: how many tiers, and what names?
7. **Stats**: which stats do bosses, creatures and weapons have?
8. **Seeds**: should the same request and seed always give the same sprite? (Recommended: yes. It helps with debugging, saving games and multiplayer.)
9. **JSON**: OK to use JSON for the rule files? (Recommended: yes. It is simple, widely supported and fast to load. YAML is easier to type but slower and trickier to read correctly.)

## Easy-to-miss things (for beginners)

- **Versioning rules**: when a rule file changes, old saved sprites can change too. Storing the version number in every output prevents surprises.
- **Validation messages**: clear errors for bad rule files save hours. For example: "tree.json: leaves.color.default is not inside allowed_hue".
- **Caching**: the same request and seed always gives the same result, so results can be saved and reused instead of generated again.
- **Thread safety**: keep rules read-only after loading (see `01-how-it-works.md`).
- **Licenses**: colors or ideas from the internet are usually fine, but images copied from the internet usually are not.
- **Start small**: make one sprite type perfect before adding many.