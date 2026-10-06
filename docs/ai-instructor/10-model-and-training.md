# 10 — The Model and Training

## Chosen model: Gemma 4 (Suggestion, based on your preference)

| Model | Size | Memory to run (approx.) | When to use |
|-------|------|------------------------|-------------|
| **Gemma 4 E2B** | about 2 billion *effective* parameters | about 1.5 GB at 4-bit, about 4 GB at full precision | Developing on a weak computer, quick tests |
| **Gemma 4 E4B** ⭐ | about 4 billion *effective* parameters (the stored file is larger) | about 3–5 GB at 4-bit, about 15 GB at full precision | **Main model.** Better reasoning, so better decisions |
| Gemma 4 26B A4B / 31B | much bigger | 16 GB+ even at 4-bit | Later, with a powerful computer or cloud |

Memory numbers come from public guides and model pages; we will measure them ourselves.

Facts from Google's Gemma 4 model card and docs:
- **Thinking**: every Gemma 4 model has a configurable thinking mode.
- **Context window**: the small models (E2B, E4B) can read up to 128K tokens, which is far more than our rules and requests need.
- **System prompt** and **function calling** are built in. The system prompt is where our theme and type rules go.
- **Multimodal**: E2B and E4B also accept images. This could later let the AI *look* at a generated sprite and check it (Suggestion for later, not Scope 1).

"Effective parameters" means the model runs about as fast as a 2B or 4B model, even though it stores some extra special layers.

Because the model is a **setting in a config file**, you can develop with E2B now and switch to E4B (or bigger) on a better computer or in the cloud, without changing code.

## No "frontend" to strip

A model like Gemma is just two things:
- **Weights**: a big file of numbers learned during training. This is the model's "brain".
- **Tokenizer**: turns text into numbers and back.

The chat window you see in AI apps is a separate program, not part of the model. Our C++ tool **is** our frontend: it builds the prompt from the rule files, runs the model with llama.cpp, separates the thinking from the answer and validates the answer.

## Making it fast without making it weak

We keep the full model's reasoning and optimize **our code** around it. The list is in `01-how-it-works.md` ("Performance").

Shrinking with **quantization** is optional. It means storing each number with fewer bits, and it does not need retraining:

```
Read this: file size ≈ parameters × bits per parameter ÷ 8   (8 bits = 1 byte)
8 billion stored × 16 ÷ 8 ≈ 16 GB    (full precision)
8 billion stored ×  4 ÷ 8 ≈  4 GB    (4-bit)
```

Google publishes official 4-bit Gemma 4 versions trained to lose very little quality (called **QAT**, quantization-aware training). We will test the quality with and without it and decide based on the results.

## How training works (beginner version)

**Fine-tuning** means taking a model that already understands language and reasoning, then training it a little more on our own examples, so it becomes an expert at our job.

A **training example** is an input plus the output we want:
- **Input**: the rules (tree + Christmas) and a request
- **Output**: good thinking, then a good Generation Instruction (JSON)

The model sees many examples and slowly adjusts its weights so its outputs look more like the good ones.

**LoRA (Suggestion):** instead of changing all the weights, we train a small **add-on** (an "adapter") on top of the model, while the original weights stay frozen. It needs much less memory and time, so it fits Kaggle's free tier. After training, the adapter is merged into the model.

## Where the training examples come from (you have none yet)

1. **Hand-written**: you write 20–50 excellent examples. These matter most for quality and style.
2. **Generated**: the M3 model (Gemma 4 with no fine-tuning) writes many examples from the rule files. The validator throws away any that break the rules, and you spot-check the rest.
3. **Approved results**: once the tool runs, every approved asset becomes an example. Rejected ones, with the reason, teach what to avoid.

For one narrow task like this, a few hundred to a few thousand good examples is usually enough. We will measure with the fixed test set (see `09-questions-and-gaps.md`).

## Kaggle (for learning and experiments)

You plan to use Kaggle's free tier to learn and experiment, not to run the finished tool. Two options:
- **TPU v5e-8** (8 chips, 128 GB of memory in total): KerasHub + JAX. LoRA fine-tuning of E2B or E4B fits easily.
- **T4 GPU**: Hugging Face transformers + PEFT. Google's own Gemma 4 thinking examples run on a T4.

Free hours are limited per week and sessions end after a few hours, so save the trained adapter at the end of every session.

## From training to the C++ tool

```
Kaggle: fine-tune (LoRA) → merge adapter → save weights
   → convert to GGUF (llama.cpp script) → (optional) quantize
   → load in our C++ tool with llama.cpp → thinking on, JSON grammar on the answer
```

## What the model does NOT do

It thinks and writes instructions and animation plans. It does **not** draw images. Drawing the snowy trees is Scope 2.