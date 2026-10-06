# AI Instructor — Read This First

This folder is the plan for **Scope 1** of the AI Image Creator Project: the **AI Instructor**.
It is written for a beginner. Read the files in number order. Each file is short and covers one topic.

The **project rules** (what the project is, requirements, coding guidelines) are in [`../project/`](../project/). AI coding tools read them through `AGENTS.md` and `CLAUDE.md` in the repo root.

## Version

**v3 (2026-10-06).** Changes after your feedback:
- The AI Instructor is a **thinking** neural network: **Gemma 4 E4B** as the main model, and **E2B** as the lighter option. The model is a config setting, so it can be switched later.
- The model may be heavy. We optimize **our code** around it instead of stripping the model down.
- **Size is decided by developers per sprite** so it fits the tile grid. It is not locked for every tree.
- The Image Creator is a **separate tool**. The game only receives approved results, through an **Asset Package**.
- The goal is a **good** engine, not the cheapest one.

Still true from v2: the rule files are **guardrails**, a C++ **validator** enforces them, results are **dynamic**, and assets are **generated ahead of time** and released on a schedule.

## The 3 scopes of the project

| # | Scope | What it does | Status |
|---|-------|--------------|--------|
| 1 | **AI Instruction Generator** ("AI Instructor") | Reads the rules and a request, thinks, then writes the exact instruction the Image Generator needs. It also **plans** the animation. | **Current scope** |
| 2 | Image Generator | Takes the instruction and draws the image. | Later |
| 3 | Animator | Joins the generated images and animates them, following the animation plan from Scope 1. | Later |

## Words used in these documents (glossary)

- **Sprite**: a 2D game image, such as a tree, a soil tile, a boss, a jungle creature, a crafting item, or a crafting resource.
- **Tile**: one square of the game grid (for example 32×32 pixels). Sprite sizes are whole numbers of tiles.
- **Sprite Type**: a kind of sprite, such as `tree` or `boss`. Every type has one **Sprite Type file** with its rules.
- **Theme file**: one file with the art style shared by all sprites, so everything looks like the same game.
- **Event file**: rules for a game event, such as Christmas.
- **Guardrails**: the rules the AI must stay inside. The AI is free inside them.
- **Model**: the neural network. Here it is a **language model** that thinks and then writes JSON.
- **Thinking**: the model's step-by-step reasoning before its answer. Saved in the logs.
- **Fine-tuning**: extra training that turns a general model into an expert at one job.
- **Validator**: plain C++ code that checks the AI's output against the rules.
- **Request**: what asks for assets. Example: "200 trees for Christmas, size 1×2 tiles, leaves color = auto".
- **auto**: "AI, you choose this, inside the rules".
- **Generation Instruction**: the AI Instructor's output for the Image Generator (Scope 2).
- **Animation Plan**: the AI Instructor's output for the Animator (Scope 3).
- **Asset Package**: the folder of approved results that the game reads.
- **Logs**: records of everything the AI did. They are used to find problems and to train the next model version.

## File map

| File | Topic |
|------|-------|
| `01-how-it-works.md` | The big picture: thinking model + guardrails + validator, separate tool, performance |
| `02-rule-files.md` | What to write in the rule files (theme, sprite types, events), and why |
| `03-request-and-output.md` | What goes in, what comes out, and how results reach the game |
| `04-animation-planning.md` | How the AI plans animation now, so Scope 3 can run it later |
| `05-full-sprites-stats-rarity.md` | Creating whole sprites with stats and rarity under the rules |
| `06-logging.md` | What to log, and how logs become training data |
| `07-libraries.md` | Which libraries to use and which to avoid |
| `08-roadmap.md` | Build order, in small steps you can test |
| `09-questions-and-gaps.md` | Things still to decide, and things that are easy to miss |
| `10-model-and-training.md` | Gemma 4 E2B/E4B, thinking, and how to train on Kaggle |

## Status of this plan

This is a **draft**. Anything marked **Suggestion** is my proposal, not a decision.