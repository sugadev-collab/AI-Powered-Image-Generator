# 09 — Open Questions and Easy-to-Miss Things

## Questions for you (decisions needed)

1. **What will the Image Generator (Scope 2) be?** It is still the biggest open decision. Snowy trees that all look different need the Image Generator to *draw* the variety the AI Instructor describes. Options:
   - **a. Procedural drawing**: C++ code draws each part from shapes, colors and textures. Fast, consistent look.
   - **b. Part assembly**: drawn parts (trunks, leaf shapes, snow caps) are layered and recolored. Fast, the most consistent look.
   - **c. AI image model** (a neural network that draws, such as Stable Diffusion). The most variety, but it needs a strong GPU, and keeping one look across the game is harder.
   - It can also be a mix. Scope 1 works with all of them, but the instruction details differ a little.
2. **Art style and tile size**: pixel art? What tile size (16, 32 or 64 pixels)?
3. **Training in Python**: is it OK that the Kaggle training notebooks are Python? The tool itself stays C++ (see `07-libraries.md`).
4. **Your computer now**: what CPU, how much RAM, and is there a graphics card (GPU)? This decides whether you develop with E2B or E4B locally.
5. **Review**: should a person approve every asset, or should assets that pass the validator be approved automatically with spot checks?
6. **Rarity**: how many tiers, and what names?
7. **Stats**: which stats do bosses, creatures and weapons have?
8. **Web3 / NFTs**: OK to decide later? (See `01-how-it-works.md`.)

## Easy-to-miss things (for beginners)

- **Model license**: check the license on Google's official Gemma 4 model card before you release anything. Websites sometimes describe it differently, so trust the model card.
- **Data quality beats data amount**: 50 excellent examples teach more than 5,000 sloppy ones.
- **A fixed test set**: keep about 50 requests that are never used for training. Run every new model version on them to see if it really got better.
- **Versioning**: store rule versions **and** the model version with every asset, so old assets can always be explained.
- **Kaggle limits**: free GPU and TPU time is limited per week, and sessions end after a few hours. Save your work at the end of every session. Kaggle is for learning and experiments, not for running the finished tool.
- **Thinking takes time**: more thinking gives better choices but slower batches. Tune the thinking budget per sprite type.
- **Internet content**: colors and ideas from the internet are usually fine, but copying images usually is not.
- **Start small**: make one sprite type and one event work well before adding more.