# 07 — Libraries

Your rule: small libraries only, nothing large or unnecessary. All of these are free.

There are two separate parts:
- **The engine** (C++): runs the model, applies rules, validates and logs. This is what we build and ship.
- **Training** (Python, on Kaggle): only used to train the model. It is **not** part of the engine and is never shipped. Training tools only exist in Python, so this is the one place we cannot use C++.

## Engine (C++)

| Need | Library | Why |
|------|---------|-----|
| Build system | **CMake** | The standard way to build C++ on Windows, Linux and Mac. |
| Run the neural network | **llama.cpp** | Runs small language models fast on a normal CPU (GPU optional). Supports 4-bit models and grammar-forced JSON output (see `01-how-it-works.md`). Written in C/C++ and used directly from C++. It is the biggest library we use, but it is the core of the AI. |
| Read rule files (JSON) | **nlohmann/json** | One header file, the easiest C++ JSON library for a beginner. |
| Logging | **Our own small logger** (about 100–150 lines), or **spdlog** | Gives exactly the features in `06-logging.md`. |
| Unit tests | **doctest** | One header file. Light and fast to compile. |
| Internet (research step) | **libcurl** | The standard HTTP library. It is C code called directly from C++, so we mark it with a "this is C code" comment. |

## Training (Python, on Kaggle only)

| Need | Library | Why |
|------|---------|-----|
| Fine-tune on TPU | **KerasHub + JAX** | Google's official way to fine-tune Gemma on TPUs, which matches Kaggle's TPU v5e-8. |
| Alternative on GPU | **Hugging Face transformers + PEFT** | The most common tools, if we use a GPU or a non-Gemma model. |
| Convert for C++ | **llama.cpp conversion scripts** | Turn the trained model into a GGUF file that llama.cpp can load, then shrink it to 4-bit. |

## About C inside C++

You are mostly right: C++ can call C code and C libraries directly, and libcurl is an example. But not every C file is valid C++, because a few C features are written differently in C++. In practice this rarely matters. We keep C to places like libcurl and mark it with a comment, as you asked.

## Avoid in the engine

| Library | Why not |
|---------|---------|
| PyTorch, TensorFlow | Large, and built for training. llama.cpp is much smaller for running a finished model. (They are fine inside Kaggle training notebooks.) |
| Boost (whole library) | Very large. The standard library covers what we need. |
| Paid AI APIs | Your budget is 0 USD. |