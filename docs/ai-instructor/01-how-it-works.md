# 01 — How the AI Instructor Works

## The big idea

The AI Instructor is a **thinking neural network**: a language model that reasons before it answers. It writes instructions for the Image Generator.

- **It thinks.** Before writing an instruction, the model reasons step by step. For example: "It's Christmas, this is a tree, the leaves color is `auto`, the theme is cozy jungle, so I'll pick a deep pine green that still shows snow clearly." This reasoning is what helps it make the right decision.
- **It is dynamic.** The same request can give a different result every time, so every tree, boss or creature can be unique.
- **The rule files are guardrails.** They are not the decision maker. They tell the AI what it must keep and where its limits are, so it does not invent things you don't want.

**Chosen model (Suggestion, based on your preference):** **Gemma 4 E4B** as the main model, because it reasons better. **Gemma 4 E2B** is the lighter option while your computer is not powerful. Both have a built-in thinking mode. The model is a **setting** in a config file, so we can switch between E2B, E4B or a bigger model later without changing code. See `10-model-and-training.md`.

## The Image Creator is a separate tool

The Image Creator Tool (AI Instructor + Image Generator + Animator) is its **own program**. It does **not** run inside the game. The game only receives the **finished, approved results**.

```
┌──────────── Image Creator Tool (separate program) ────────────┐
│  AI Instructor  →  Image Generator  →  Animator  →  Review     │
│    (Scope 1)         (Scope 2)          (Scope 3)               │
└───────────────────────────────┬────────────────────────────────┘
                                │  export: only approved results
                                ▼
                        Asset Package (folder)
                  images + animation data + manifest
                                │
                                ▼
                    Game loads each asset when its time comes
```

Why this is good:
- The model can be heavy. It never slows down the game, because the game never runs it.
- Players never need a powerful computer. Only the machine that generates assets does.
- The game stays simple. It only reads files.

## Who decides what

| Who | Decides |
|-----|---------|
| **Developers** (rule files and requests) | What is **static**, what **may change**, and the limits of each change. Developers also decide each sprite's **size**, so it fits the right tiles. |
| **The AI** (thinking neural network) | Everything that is allowed to change, such as colors, textures, shapes, small details, animation and stats. It is fully dynamic inside the limits. |
| **The Validator** (plain C++ code) | Whether the AI's output really follows the rules. If not, the output is fixed or the AI tries again. |

**About size:** size is **not** fixed for every tree. Sprites must fit the game's **tile grid** (for example 32×32 pixel tiles). A developer can ask for an exact size, such as "this tree fills 1×2 tiles", and the AI then follows it strictly. Or the developer can set `auto`, and the AI picks a size inside the limits in the rule file. Details are in `02-rule-files.md`.

**Why a validator?** Thinking makes the AI much better, but it can still make mistakes, such as picking a color outside the allowed range. The validator is a cheap, fast safety net. It makes sure the guardrails are always followed instead of only hoping the AI follows them.

## Example: a Christmas event

1. A developer adds an event file, `christmas.json`: snow on top surfaces is allowed, white and red are added to the palette, and lights and ornaments are allowed as decorations.
2. A request asks for 200 trees for the Christmas event, each 1×2 tiles.
3. For each tree, the AI **thinks** about the theme, the event and the limits, then writes a **different** instruction: different snow amounts, light colors, ornament positions and sway animations. Every tree keeps the 1×2 tile size the developer asked for.
4. The validator checks every instruction.
5. The images are drawn (Scope 2), reviewed, and exported to the game, set to appear on 20 December.

## The pipeline (the steps)

A pipeline is a list of steps where the output of one step is the input of the next.

```
Request (+ optional event)
   │
1. Load rules ......... theme + sprite type + event (loaded once, kept in memory)
2. Merge .............. request values + defaults. Strict values are applied and cannot change.
3. Research ........... (optional) look up new information on the internet and add it as context
4. Think .............. the model reasons in free text about the choices (saved to the log)
5. Answer ............. the model writes the Generation Instruction + Animation Plan as JSON,
   │                    filling every "auto" value and every free detail
6. Validate ........... C++ checks every rule. Small mistakes are fixed (for example, a value is
   │                    clamped back into its limit). Big mistakes: ask the model again (up to N tries).
7. Variety check ...... (optional) compare with assets already made; if too similar, generate again
8. Output + log
   │
   ... later: draw (Scope 2) → animate (Scope 3) → review → export to the game
```

## How thinking works

With thinking mode on, the model writes its answer in two parts:
1. **Thinking**: free text where it reasons, compares options and checks itself.
2. **Answer**: the final JSON instruction.

We keep both. The **answer** goes to the Image Generator. The **thinking** goes to the log, so you can read **why** the AI chose something. This makes improving the AI much easier.

Thinking costs time, because the model writes more text. So we set a **thinking budget** (how much it may think) per sprite type in the rule files. For example, bosses can think a lot, and soil tiles only a little.

## How the model is forced to write the right format

A language model writes text one small piece at a time. Each piece is called a **token**.

We can give the model a **grammar**, which is a description of the exact JSON shape we accept. While the model writes the **answer**, any token that would break the grammar is blocked. So the answer is **always valid JSON with the right fields**. This is called **constrained decoding**, and llama.cpp supports it (see `07-libraries.md`). The thinking part stays free text, so the grammar does not limit how the model reasons.

The grammar checks the **shape**. The validator checks the **content**, for example "is this green inside the allowed hue range?".

## Dynamic by default, and how we still debug

By default **every run uses a new random seed** (the number behind the model's random choices), so results differ.

**Suggestion:** still **write down** the seed and model version in the log. We never reuse it normally. But if one tree comes out broken, we can recreate that exact tree and see why. Writing it down does not make the AI any less dynamic.

How adventurous the AI is can be tuned with **temperature**:
- low (for example 0.3): safe choices, results look similar
- high (for example 1.0): more variety, more surprises, more mistakes for the validator to catch

Temperature can be set per sprite type in the rule files.

## Internet access ("learning something new")

Assets are made **ahead of time** in a separate tool, so the AI can use the internet freely without slowing down the game.

There are two kinds of "learning", and both are useful:

1. **Looking things up (every run).** The research step searches the internet, for example for "traditional Christmas tree decorations". It saves the results in a cache and gives a short summary to the model. The model can then think about it in this run.
2. **Really learning (when we retrain).** Approved outputs, rejected outputs and logs become new training examples. We fine-tune again, and the new model version is better. See `10-model-and-training.md`.

Important: a model does **not** change itself while it runs. Its "brain" (the weights) only improves when we train it.

## Timing and time order

Because generation is not live, time order is a **schedule**:
- Every request can have a `release_at` date and time: when the asset should appear in the game.
- The tool works on requests early, earliest `release_at` first, so the soonest assets are ready first.
- Approved assets are exported with their `release_at` in the package **manifest** (a list of every asset and when it appears). The game loads each one when its time comes.

Timing **inside** an animation (keyframes) is still planned by the AI. See `04-animation-planning.md`.

## Performance: the model can be heavy, so we optimize our side

A bigger thinking model gives better decisions, so we don't strip it down. Instead we make **our code** around it fast:

1. **Load the model once.** Loading takes seconds, so we load it at startup and run many requests back to back.
2. **Reuse the shared part of the prompt.** Every request in a batch starts with the same theme and type rules. llama.cpp can keep that part already processed and reuse it (this is called **prompt caching**), so only the new part of each request is processed.
3. **Send only relevant rules.** A tree request gets the tree rules, not the boss rules. Shorter prompts are faster.
4. **Thinking budget** per sprite type (see above).
5. **Use a graphics card (GPU) when there is one.** llama.cpp moves the model onto the GPU automatically if one exists, and falls back to the CPU otherwise.
6. **Valid JSON the first time**, thanks to the grammar, so no time is wasted on broken answers.

One correction to "we can't shrink a trained model": we actually **can**, without retraining. This is called **quantization**: storing the model's numbers with fewer bits. Google even publishes official 4-bit versions of Gemma 4. It is **optional**. We will compare quality, and only use it if the result stays good. It would let E4B run on a smaller computer.

Rough speed (to be measured): on a CPU only, E2B and E4B write about 2–8 tokens per second. With thinking, one sprite may take a minute or more. On a GPU it takes seconds. This is fine, because generation happens ahead of time.

## Web3 and NFTs (your question)

"Will my game be a Web3 game?" is your choice, and it does not need to be decided now. The AI can make every asset unique either way.

**Suggestion:** decide later. Meanwhile, give every generated asset a **unique ID** and store its full instruction with it. That is exactly the "metadata" an NFT would need later, so nothing is lost if you choose Web3.