# 08 — Roadmap (build order)

Small steps. Each step ends with something you can run and test. We start with **one** Sprite Type (tree) and **one** event (Christmas).

| Step | Goal | Done when |
|------|------|-----------|
| **M0** | Project setup: CMake, folder layout, doctest, small logger | `build` runs and one test passes |
| **M1** | Rule files (`theme.json`, `types/tree.json`, `events/christmas.json`) and a C++ loader | A test prints the merged tree + Christmas rules |
| **M2** | **Validator** in C++ (the guardrail) | Hand-written good instructions pass; hand-written bad ones fail with clear messages |
| **M3** | **First dynamic AI**: run a ready-made small model with llama.cpp. The prompt contains the rules and the request; the grammar forces the JSON shape | 10 Christmas trees come out different and all pass the validator. No training needed yet. |
| **M4** | Logging (Run, Review, Runtime logs) | Every generated tree has a full log entry |
| **M5** | Batch runner, review list (approve or reject) and schedule (`release_at`) | 200 trees are generated, reviewed and listed by release time |
| **M6** | Training dataset: hand-written + generated + approved examples | A clean dataset file of a few hundred examples |
| **M7** | First fine-tune on Kaggle TPU → convert to GGUF → 4-bit | The fine-tuned model beats the M3 model on a fixed test set |
| **M8** | Animation plans (AI picks the values, C++ computes keyframes) | `idle_sway`, `lights_twinkle` and `grow` plans pass validation |
| **M9** | Rarity, stats, full sprites (one weapon type, one boss type), variety check | Stats always fit the budget; no two bosses are too similar |
| **M10** | Research step: internet search → cache → context for the model | A request uses fresh research and the log shows what was used |
| **M11** | More Sprite Types: soil, creatures, crafting items, resources | Each new type works by adding a rule file (and, later, some training examples) |

**Key point:** at M3 you already have a working, dynamic AI, before any training. Training (M7) makes it better at your style.

## Suggested code layout (to create at M0)

```
docs/ai-instructor/     (these documents)
rules/                  (theme, rarity, types/, events/)
src/instructor/         (the AI Instructor engine, C++)
training/               (Kaggle notebooks and dataset scripts, Python)
models/                 (GGUF model files, not committed to Git because they are large)
tests/                  (doctest tests)
CMakeLists.txt
```