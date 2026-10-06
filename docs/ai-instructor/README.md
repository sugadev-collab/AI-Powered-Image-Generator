# AI Instructor — Read This First

This folder is the plan for **Scope 1** of the AI Image Creator Project: the **AI Instructor**.
It is written for a beginner. Read the files in number order. Each file is short and covers one topic.

## Version

**v2 (2026-10-06).** Reworked after your feedback:
- The AI Instructor is now a **small neural network** (a small language model). It is fully dynamic, so the same request can give a different result every time.
- The rule files are **guardrails**. They control the AI so it does not invent things you don't want. They do not make the decisions.
- Assets are **generated ahead of time**, validated, and then added to the game automatically when their time comes. There is no live generation inside the game for now.

## The 3 scopes of the project

| # | Scope | What it does | Status |
|---|-------|--------------|--------|
| 1 | **AI Instruction Generator** ("AI Instructor") | Reads the rules and a request, then writes the exact instruction the Image Generator needs. It also **plans** the animation. | **Current scope** |
| 2 | Image Generator | Takes the instruction and draws the image. | Later |
| 3 | Animator | Joins the generated images and animates them, following the animation plan from Scope 1. | Later |

Scope 1 ends when the AI Instructor outputs a **Generation Instruction** (what to draw) and an **Animation Plan** (how it will move later).

## How the whole tool will be used (big picture)

```
1. Generate early ... AI Instructor writes instructions + animation plans   (Scope 1)
                      Image Generator draws the images                      (Scope 2)
                      Animator builds the animations                        (Scope 3)
2. Validate ......... automatic checks, then optional human review
3. Schedule ......... every approved asset gets a release time (e.g. the Christmas event)
4. Integrate ........ the game loads the asset automatically when its time comes
```

## Words used in these documents (glossary)

- **Sprite**: a 2D game image, such as a tree, a soil tile, a boss, a jungle creature, a crafting item, or a crafting resource.
- **Sprite Type**: a kind of sprite, such as `tree` or `boss`. Every type has one **Sprite Type file** with its rules.
- **Theme file**: one file with the art style shared by all sprites, so everything looks like the same game.
- **Event file**: rules for a game event, such as Christmas. It changes what the AI may do during that event (for example, snow is allowed).
- **Guardrails**: the rules the AI must stay inside. The AI is free inside them.
- **Model**: the neural network. Here it is a small **language model**, a neural network that reads and writes text (we make it write JSON).
- **Fine-tuning**: extra training that turns a general model into an expert at one job (writing our instructions).
- **Validator**: plain C++ code that checks the AI's output against the rules.
- **Request**: what asks the AI Instructor for assets. Example: "200 trees for Christmas, leaves color = auto".
- **auto**: "AI, you choose this, inside the rules".
- **Generation Instruction**: the AI Instructor's output for the Image Generator (Scope 2).
- **Animation Plan**: the AI Instructor's output for the Animator (Scope 3).
- **Logs**: records of everything the AI did. They are used to find problems and to train the next model version.

## File map

| File | Topic |
|------|-------|
| `01-how-it-works.md` | The big picture: neural AI + guardrails + validator, internet, timing, Web3 |
| `02-rule-files.md` | What to write in the rule files (theme, sprite types, events), and why |
| `03-request-and-output.md` | What goes in, and what the AI Instructor sends out |
| `04-animation-planning.md` | How the AI plans animation now, so Scope 3 can run it later |
| `05-full-sprites-stats-rarity.md` | Creating whole sprites with stats and rarity under the rules |
| `06-logging.md` | What to log, and how logs become training data |
| `07-libraries.md` | Which libraries to use and which to avoid |
| `08-roadmap.md` | Build order, in small steps you can test |
| `09-questions-and-gaps.md` | Things still to decide, and things that are easy to miss |
| `10-model-and-training.md` | Which model, how to train it on Kaggle, and your Gemma question |

## Status of this plan

This is a **draft**. Anything marked **Suggestion** is my proposal, not a decision.