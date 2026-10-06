# AGENTS.md — Rules for AI Coding Assistants

This file is for AI coding tools (Codex, Claude Code and others). **Read it before doing anything in this repository.** The full project rules are in [`docs/project/`](docs/project/).

## Repository boundary

- This repository, `sugadev-collab/AI-Powered-Image-Generator`, is the **only** repository in scope for this project.
- Never read from, change, or push to any other repository of the owner.
- Do not change files the owner did not ask you to change.

## What this project is (short)

An AI-powered, dynamic tool that creates and animates 2D game images (sprites) from instructions written by an AI. It has 3 scopes:

1. **AI Instructor** (current scope): a thinking neural network that writes the instructions the Image Generator needs, and plans the animation.
2. **Image Generator**: draws the images from those instructions.
3. **Animator**: joins and animates the images.

The tool runs **separately** from the game. The game only receives the finished, approved results.

Details: [`docs/project/01-overview.md`](docs/project/01-overview.md) and [`docs/project/02-ai-instructor-requirements.md`](docs/project/02-ai-instructor-requirements.md).
The current design plan (a draft) is in [`docs/ai-instructor/`](docs/ai-instructor/README.md).

## Coding rules (must follow)

Full list: [`docs/project/03-coding-guidelines.md`](docs/project/03-coding-guidelines.md).

- Write **pure C++**. C is allowed only where it is much faster, and must be marked with a "this is C code" comment.
- Put a **short explanation comment above every function**.
- **Explain every mathematical formula** above its code.
- Do not comment every line. Put longer explanations at the **top of the file**, marked "Read this before reading the code".
- Keep code clean, structured and reusable. If reuse would hurt performance, duplicating code is fine: **performance comes first**.
- Libraries are allowed, but nothing big or unnecessary.

## Working with the owner

Full notes: [`docs/project/04-owner-and-working-style.md`](docs/project/04-owner-and-working-style.md).

- The owner is learning AI, C++ and graphics while building this. **Explain things in a beginner-friendly way.**
- Mark your own proposals as **Suggestion**. Do not present them as decisions.
- When something is unclear, **ask the owner** instead of guessing.