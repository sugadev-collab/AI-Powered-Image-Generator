# 09 — Open Questions and Easy-to-Miss Things

## Questions for you (decisions needed)

1. **What will the Image Generator (Scope 2) be?** It is still the biggest open decision. Snowy trees that all look different need the Image Generator to *draw* the variety the AI Instructor describes. Options:
   - **a. Procedural drawing**: C++ code draws each part from shapes, colors and textures. Fast, free, consistent look.
   - **b. Part assembly**: drawn parts (trunks, leaf shapes, snow caps) are layered and recolored. Fast, free, the most consistent look.
   - **c. AI image model** (a neural network that draws, such as Stable Diffusion). The most variety, but heavy, it needs a GPU, and keeping one look across the game is hard.
   - It can also be a mix. Scope 1 works with all of them, but the instruction details differ a little.
2. **Art style and sprite sizes**: pixel art? Usual sizes (16×16, 32×32, 64×64)?
3. **Training in Python**: is it OK that the Kaggle training notebooks are Python? The engine stays pure C++ (see `07-libraries.md`).
4. **Your computer**: what CPU, how much RAM, and is there a graphics card (GPU)? This decides the model size we can run comfortably.
5. **Review**: should a person approve every asset, or should assets that pass the validator be approved automatically with spot checks?
6. **Rarity**: how many tiers, and what names?
7. **Stats**: which stats do bosses, creatures and weapons have?
8. **Web3 / NFTs**: OK to decide later? (See `01-how-it-works.md`.)

## Easy-to-miss things (for beginners)

- **Model licenses**: Gemma has its own "Gemma Terms of Use" (commercial use is allowed, with a list of prohibited uses). Qwen and SmolLM use Apache 2.0. Check the license of the exact model before releasing the game.
- **Data quality beats data amount**: 50 excellent examples teach more than 5,000 sloppy ones.
- **A fixed test set**: keep about 50 requests that are never used for training. Run every new model version on them to see if it really got better.
- **Versioning**: store rule versions **and** the model version with every asset, so old assets can always be explained.
- **Kaggle limits**: free TPU time is limited per week and sessions end after a few hours. Save your work at the end of every session.
- **Internet content**: colors and ideas from the internet are usually fine, but copying images usually is not.
- **Start small**: make one sprite type and one event work well before adding more.