# 07 — Libraries

Your rule: libraries are fine, but nothing big or unnecessary. The goal is a **good** engine, not the cheapest one, so we pick the right tool for each job and keep the rest small.

There are two separate parts:
- **The tool** (C++): runs the model, applies rules, validates, logs and exports. This is what we build.
- **Training** (Python, on Kaggle): only used to train the model. It is **not** part of the tool. Training tools only exist in Python, so this is the one place we cannot use C++.

## The tool (C++)

| Need | Library | Why |
|------|---------|-----|
| Build system | **CMake** | The standard way to build C++ on Windows, Linux and Mac. |
| Run the neural network | **llama.cpp** | Runs Gemma 4 on CPU or GPU (it uses the GPU automatically when one exists). Supports thinking models, grammar-forced JSON answers and prompt caching (see `01-how-it-works.md`). Written in C/C++ and used directly from C++. It is the biggest library we use, but it is the heart of the AI. |
| Read rule files (JSON) | **nlohmann/json** | One header file, the easiest C++ JSON library for a beginner. If JSON reading ever becomes slow, **simdjson** is a faster swap. |
| Logging | **Our own small logger** (about 100–150 lines), or **spdlog** | Gives exactly the features in `06-logging.md`. |
| Unit tests | **doctest** | One header file. Light and fast to compile. |
| Internet (research step) | **libcurl** | The standard HTTP library. It is C code called directly from C++, so we mark it with a "this is C code" comment. |

## Training (Python, on Kaggle only)

| Need | Library | Why |
|------|---------|-----|
| Fine-tune on TPU | **KerasHub + JAX** | Google's official way to fine-tune Gemma on TPUs, which matches Kaggle's TPU v5e-8. |
| Alternative on GPU | **Hugging Face transformers + PEFT** | The most common tools, and Google's Gemma 4 thinking examples use them on a free Kaggle/Colab T4 GPU. |
| Convert for C++ | **llama.cpp conversion scripts** | Turn the trained model into a GGUF file that llama.cpp can load. |

## About C inside C++

You are mostly right: C++ can call C code and C libraries directly, and libcurl is an example. But not every C file is valid C++, because a few C features are written differently in C++. In practice this rarely matters. We keep C to places like libcurl and mark it with a comment, as you asked.

## Avoid in the tool

| Library | Why not |
|---------|---------|
| PyTorch, TensorFlow | Built for training and much larger than llama.cpp for running a finished model. They are fine inside the Kaggle training notebooks. |
| Boost (whole library) | Very large. The standard library covers what we need. |
| A cloud AI API as the main model | It would tie the tool to someone else's service and cost per request. Running our own fine-tuned Gemma keeps control with you. (It can still be used for testing or research if you want.) |