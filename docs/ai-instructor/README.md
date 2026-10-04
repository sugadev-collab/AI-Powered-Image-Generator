# AI Instructor — Read This First

This folder is the plan for **Scope 1** of the AI Image Creator Project: the **AI Instructor**.
It is written for a beginner. Read the files in number order. Each file is short and covers one topic.

## The 3 scopes of the project

| # | Scope | What it does | Status |
|---|-------|--------------|--------|
| 1 | **AI Instruction Generator** ("AI Instructor") | Reads our rules and a request, then outputs the exact instruction the Image Generator needs. It also **plans** the animation. | **Current scope** |
| 2 | Image Generator | Takes the instruction and draws the image. | Later |
| 3 | Animator | Joins the generated images and animates them, following the animation plan from Scope 1. | Later |

Scope 1 ends when the AI Instructor outputs two things:
1. a **Generation Instruction** (what to draw), and
2. an **Animation Plan** (how it will move later).

It does not draw anything and it does not run any animation.

## Words used in these documents (glossary)

- **Sprite**: a 2D game image, such as a tree, a soil tile, a boss, a jungle creature, a crafting item, or a crafting resource.
- **Sprite Type**: a kind of sprite, such as `tree` or `boss`. Every type has one **Sprite Type file** with its default rules.
- **Theme file**: one file with the art style shared by all sprites, so everything looks like it belongs to the same game.
- **Request**: what another part of the game sends to the AI Instructor. Example: "a tree, size fixed, leaves color = auto".
- **auto**: a value in a request that means "AI, you choose this, but stay inside the rules".
- **Generation Instruction**: the AI Instructor's output for the Image Generator (Scope 2).
- **Animation Plan**: the AI Instructor's output for the Animator (Scope 3).
- **Seed**: a number that controls the AI's random choices. The same request with the same seed always gives the same result. This makes bugs easy to repeat and fix.
- **Decision Log**: a record of every choice the AI made and why.

## File map

| File | Topic |
|------|-------|
| `01-how-it-works.md` | The big picture: the steps the AI Instructor follows, and my opinion on your ideas |
| `02-rule-files.md` | What to write in the default rule files, and why each part matters |
| `03-request-and-output.md` | What the game sends in, and what the AI Instructor sends out |
| `04-animation-planning.md` | How the AI plans animation now, so Scope 3 can run it later |
| `05-full-sprites-stats-rarity.md` | Creating whole sprites with stats and rarity under the rules |
| `06-logging.md` | Logging AI decisions, searches and runtime errors without slowing the engine |
| `07-libraries.md` | Which libraries to use and which to avoid |
| `08-roadmap.md` | Build order, in small steps you can test |
| `09-questions-and-gaps.md` | Things still to decide, and things that are easy to miss |

## Status of this plan

This is a **draft**. Anything marked **Suggestion** is my proposal, not a decision. Decisions get made after you answer the questions in `09-questions-and-gaps.md`.