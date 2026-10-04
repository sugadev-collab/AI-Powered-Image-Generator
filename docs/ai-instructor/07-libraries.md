# 07 — Libraries

Your rule: small libraries only, nothing large or unnecessary. Every choice below follows that rule. All of them are free.

## Recommended (Suggestion)

| Need | Library | Size | Why |
|------|---------|------|-----|
| Build system | **CMake** | Tool, not shipped | The standard way to build C++ projects on Windows, Linux and Mac. |
| Read rule files (JSON) | **nlohmann/json** | One header file | The easiest C++ JSON library for a beginner. Rules are loaded once at startup, so its speed is more than enough. |
| Faster JSON (only if needed later) | **simdjson** | Small | Very fast reading. Only switch to it if JSON reading ever becomes a measured bottleneck. |
| Random numbers with a seed | **Our own small generator** (e.g. SplitMix64 or PCG, about 20 lines) | Tiny | The C++ standard `<random>` distributions can give different results on different compilers. Our own small code makes the same seed give the same result everywhere. |
| Logging | **Our own small logger** (about 100–150 lines) | Tiny | Gives exactly the features in `06-logging.md` and nothing more. **spdlog** is a good ready-made option if you prefer not to write it. |
| Unit tests | **doctest** | One header file | Tests check that rules work. Light and fast to compile. |
| Internet (Research Tool only) | **libcurl** | Medium, C library | The standard library for HTTP. Its C functions can be called directly from C++ with no wrapper. Mark it with a "this is C code" comment. It is only used in the Research Tool, not in the engine. |

## About C inside C++

You are mostly right: C++ can call C code and C libraries directly, and libcurl is a good example. But not every C file is valid C++. A few C features are written differently in C++ (for example, C allows converting `void*` to another pointer type without a cast). In practice this rarely matters. We will keep C to places like libcurl, and mark it with a comment as you asked.

## Optional, later only

| Need | Library | Note |
|------|---------|------|
| Free-text requests ("make a scary swamp tree") | **llama.cpp** with a small free model | Runs locally on a normal CPU, no cost. Only in the Research Tool or an offline step, never in the fast path. |

## Avoid for this scope

| Library | Why not |
|---------|---------|
| PyTorch, TensorFlow, ONNX Runtime | Large. Built for training and running neural networks, which Scope 1 does not need. |
| Boost (whole library) | Very large. We only need small pieces, and the standard library covers them. |
| Paid AI APIs | Your budget is 0 USD. |

## Training

With the rule-based approach in `01-how-it-works.md`, **no training is needed for Scope 1**. "Tuning" means editing JSON rule files. If you later want a model that learns from the logs, Kaggle's free tier can be used for small experiments, but that is optional.