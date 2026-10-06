# 10 — The Model and Training (your Gemma question)

## Short answer

**Yes, this is a common and good approach.** It is called **fine-tuning**: take a small free model that already understands language, then train it a little more on our own examples so it becomes an expert at one job, writing our instructions. The result is a small model that is very good at our task.

## One correction: a model has no "frontend" to strip

A model like Gemma is just two things:
- **Weights**: a big file of numbers learned during training. This is the model's "brain".
- **Tokenizer**: turns text into numbers and back.

The chat window you see in AI apps is a separate program, not part of the model. So there is nothing to strip. Instead we:
1. download the weights,
2. fine-tune them on our examples, and
3. run them with **our own C++ program** (using llama.cpp). That program is our "frontend": it builds the prompt from the rule files, runs the model and validates the output.

## How to keep the model small

- **Choose a small model** (about 100 million to 1 billion parameters). *Parameters* are the numbers in the weights. More parameters means smarter but slower.
- **Quantize it**: store each number with 4 bits instead of 16.

```
Read this: file size ≈ parameters × bits per parameter ÷ 8   (8 bits = 1 byte)
1 billion × 16 ÷ 8 = 2 GB      (normal)
1 billion ×  4 ÷ 8 = 0.5 GB    (4-bit, plus a little extra)
```

Removing parts of the network ("pruning") is also possible, but it is advanced, so we skip it for now.

## Candidate models (all free to download)

| Model | Size | License | Notes |
|-------|------|---------|-------|
| **Gemma 3 270M** | 270 million | Gemma Terms of Use | Tiny and very fast; Google designed it for fine-tuning on narrow tasks like ours |
| **Gemma 3 1B** | 1 billion | Gemma Terms of Use | Smarter, still small |
| **Qwen3 0.6B** | 0.6 billion | Apache 2.0 | Good at structured output |
| **SmolLM2 360M** | 360 million | Apache 2.0 | Tiny, fully open |

**Suggestion:** start with **Gemma 3 270M or 1B**. Google officially supports fine-tuning Gemma on TPUs, which matches Kaggle's TPU v5e-8. Later we can compare against Qwen3 0.6B on the same test set. New models come out often, so we will check for newer versions when we reach M3.

## How training works (beginner version)

A **training example** is an input plus the output we want:
- **Input**: the rules (tree + Christmas) and a request
- **Output**: a good Generation Instruction (JSON)

The model sees many examples and slowly adjusts its weights so its outputs look more like the good ones.

**LoRA (Suggestion):** instead of changing all the weights, we train a small **add-on** (an "adapter") on top of the model while the original weights stay frozen. It needs much less memory and time, so it fits the free tier easily. After training, the adapter is merged into the model.

## Where the training examples come from (you have none yet)

1. **Hand-written**: you write 20–50 excellent examples. These matter most for quality and style.
2. **Generated**: the M3 model (or a bigger free model) writes many examples from the rule files. The validator throws away any that break the rules, and you spot-check the rest.
3. **Approved results**: once the tool runs, every approved asset becomes an example. Rejected ones, with the reason, teach what to avoid.

For one narrow task like this, a few hundred to a few thousand good examples is usually enough. We will measure with the fixed test set (see `09-questions-and-gaps.md`).

## Kaggle TPU v5e-8

- It is free, with a limited number of hours per week (check the current quota on Kaggle).
- TPUs work best with JAX and KerasHub. Kaggle has official Gemma fine-tuning notebooks for TPUs.
- It is more than enough to LoRA-fine-tune a model of 1 billion parameters or less.
- Sessions are time-limited, so save the trained weights at the end of each session.

## From training to the C++ engine

```
Kaggle: fine-tune (LoRA) → merge adapter → save weights
   → convert to GGUF (llama.cpp script) → quantize to 4-bit
   → load in our C++ engine with llama.cpp → run with the JSON grammar
```

## "Adding an LLM makes the app heavy"

That is true for apps that run the model while the user plays. Here generation is **offline** (see `01-how-it-works.md`), so the model never runs inside the game. We still optimize:
- a small model in 4-bit (under 1 GB)
- the model loaded once, with many requests run back to back
- short prompts: only the rules relevant to the request are sent
- a fixed JSON grammar, so no time is wasted on broken outputs

## What the model does NOT do

It writes instructions and animation plans. It does **not** draw images. Drawing the snowy trees is Scope 2.