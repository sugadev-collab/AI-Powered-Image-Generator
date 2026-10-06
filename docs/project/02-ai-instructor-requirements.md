# 02 — AI Instructor Requirements (Scope 1)

These are the owner's requirements. The current design that meets them is in [`../ai-instructor/`](../ai-instructor/README.md).

## Neural network that thinks

- The AI Instructor must include a **neural network**.
- The model must be able to **think** (reason), because thinking is what makes it choose the right decision.
- It must be **fully dynamic**. Giving the same result every time for the same request is not wanted.
- Preferred models: **Gemma 4 E4B** (preferred for its stronger reasoning) or **Gemma 4 E2B**. Bigger models are possible later, on a more powerful computer or in the cloud.

## Rules control the AI

- The rule, theme and sprite type files only **control** the AI's behaviour, so it does not invent things nobody wanted.
- The game decides what is static and what can change. The AI is dynamic inside those rules.

## Requests and defaults

- The AI receives structured instructions from another part of the game, for example `size -> dimension` or `color -> auto`.
- It strictly follows every value that is defined.
- When a property is `auto`, it chooses the value itself, and that choice must fit the other strict instructions.
- Developers keep a default instruction file for each sprite type. These defaults stop the AI from inventing unwanted things.
- A request states the sprite type first, then the dimensions, then the colors and where each color goes, and more.
- Any property the request does not mention takes its default value.
- Full sprite generation must create its instructions under the rules.

## Animation planning

- While creating the instruction for the Image Generator, the AI also thinks about the animation it will make and plans it accurately.
- Animating the images (and generating animation code) is Scope 3.

## Internet

- The AI needs internet access so it can learn new things.

## Logging

- The AI's actions, searches, search results and auto choices must be logged in detail, so it can be improved by reading the logs.
- Runtime code issues should also be logged, if this is easy and does not make the tool heavier or slower.

## Time order

- Things can be put in time order, and the AI creates them for that timing.
- Assets are made ahead of time and integrated when their time comes. If time tracking inside the AI causes performance problems, timing can be handled outside the AI.

## Training

- Kaggle's free tier (for example the TPU v5e-8) can be used for training and experiments.
- This is for **learning (education)**. Kaggle is not where the finished tool runs.
- The idea is to fine-tune an existing model (Gemma as the example) instead of training one from nothing.
- Use of the model should be optimized as much as possible on our side.