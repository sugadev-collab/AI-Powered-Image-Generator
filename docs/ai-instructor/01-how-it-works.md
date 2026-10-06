# 01 — How the AI Instructor Works

## The big idea

The AI Instructor is a **small neural network**, specifically a small language model, that writes instructions for the Image Generator.

- It is **dynamic**. The same request can give a different result every time, so every tree, boss or creature can be unique.
- The rule files are **guardrails**, not the decision maker. They tell the AI what it must keep and where its limits are, so it does not invent things you don't want.

Who decides what:

| Who | Decides |
|-----|---------|
| **The game** (rule files written by developers) | What is **static** (locked), what **may change**, and the limits of each change |
| **The AI** (neural network) | Everything that is allowed to change, such as colors, textures, shapes, small details, animation and stats. It is fully dynamic inside the limits. |
| **The Validator** (plain C++ code) | Whether the AI's output really follows the rules. If not, the output is fixed or the AI tries again. |

**Why a validator?** A neural network is creative, but it can make mistakes, such as picking a color outside the allowed range. The validator is a cheap, fast safety net. It makes sure the guardrails are always followed instead of only hoping the AI follows them.

## Example: a Christmas event

1. A developer adds an event file, `christmas.json`: snow on top surfaces is allowed, white and red are added to the palette, and lights and ornaments are allowed as decorations.
2. A request asks for 200 trees for the Christmas event.
3. The AI reads the tree rules, the theme and the Christmas event, and writes 200 **different** instructions, with different snow amounts, light colors, ornament positions and sway animations. The tree size stays locked.
4. The validator checks every instruction.
5. The images are drawn (Scope 2), reviewed, and scheduled to appear on 20 December.

## The pipeline (the steps)

A pipeline is a list of steps where the output of one step is the input of the next.

```
Request (+ optional event)
   │
1. Load rules ......... theme + sprite type + event (loaded once, kept in memory)
2. Merge .............. request values + defaults. Locked values are applied and cannot change.
3. Research ........... (optional) look up new information on the internet and add it as context
4. AI generates ....... the model writes the Generation Instruction + Animation Plan as JSON,
   │                    filling every "auto" value and every free detail
5. Validate ........... C++ checks every rule. Small mistakes are fixed (for example, a size is
   │                    clamped back into its limit). Big mistakes: ask the model again (up to N tries).
6. Variety check ...... (optional) compare with assets already made; if too similar, generate again
7. Output + log
```

## How the model is forced to write the right format

A language model writes text one small piece at a time. Each piece is called a **token**.

We can give the model a **grammar**, which is a description of the exact JSON shape we accept. At every step, any token that would break the grammar is blocked. So the output is **always valid JSON with the right fields**. This is called **constrained decoding**, and llama.cpp supports it (see `07-libraries.md`).

The grammar checks the **shape**. The validator checks the **content**, for example "is this green inside the allowed hue range?".

## Dynamic by default, and how we still debug

You are right that an AI giving the same result every time is not what we want. So by default **every run uses a new random seed** (the number behind the model's random choices), and results differ.

**Suggestion:** still **write down** the seed and model version in the log. We never reuse it normally. But if one tree comes out broken, we can recreate that exact tree and see why. Writing it down does not make the AI any less dynamic.

How dynamic the AI is can be tuned with **temperature**, a model setting for how adventurous its choices are:
- low (for example 0.3): safe choices, results look similar
- high (for example 1.0): more variety, more surprises, more mistakes for the validator to catch

Temperature can be set per sprite type in the rule files. For example, bosses could be high and soil tiles low.

## Internet access ("learning something new")

Assets are made **ahead of time**, not while a player plays. So the AI can use the internet freely during generation without slowing down the game.

There are two different kinds of "learning", and both are useful:

1. **Looking things up (every run, instant).** The research step searches the internet, for example for "traditional Christmas tree decorations". It saves the results in a cache and gives a short summary to the model as extra context. The model now knows about it for this run.
2. **Really learning (when we retrain).** Approved outputs and logs become new training examples. We fine-tune again, and the new model version is better. See `10-model-and-training.md`.

Important: a model does **not** change itself while it runs. Its "brain" (the weights) only improves when we train it.

## Timing and time order

Because generation is not live, time order is a **schedule**, not part of the AI:
- Every request can have a `release_at` date and time: when the asset should appear in the game.
- The generator works on requests early, earliest `release_at` first, so the soonest assets are ready first.
- Approved assets wait in an "approved" list. The game loads them when `release_at` arrives.

Timing **inside** an animation (keyframes) is still planned by the AI. See `04-animation-planning.md`.

## Performance

Generation is offline, so the model's speed decides **how long a batch takes**, not the game's frame rate. We still optimize it:
- a **small** model (under about 1 billion parameters)
- **4-bit quantization**, which stores the model's numbers with fewer bits (about 4× smaller and faster, with a small quality loss)
- running on a normal CPU with llama.cpp, with the model loaded once and many requests run back to back

Rough estimate (to be measured): a few seconds per sprite on a normal laptop CPU. Details are in `10-model-and-training.md`.

## Web3 and NFTs (your question)

"Will my game be a Web3 game?" is your choice, and it does not need to be decided now. The AI can make every asset unique either way.

Web3 would add a blockchain wallet system, transaction fees ("gas") for creating NFTs, and extra legal and marketplace rules. These are a lot to take on while the game is not designed yet and the budget is 0 USD.

**Suggestion:** decide later. Meanwhile, give every generated asset a **unique ID** and store its full instruction with it. That is exactly the "metadata" an NFT would need later, so nothing is lost if you choose Web3.