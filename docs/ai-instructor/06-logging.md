# 06 — Logging

With a dynamic neural network, logs matter **more**, not less. They have two jobs:
1. **Find problems**: see exactly what the AI did and why a result came out wrong.
2. **Train the next model**: approved and rejected results become training examples (see `10-model-and-training.md`).

## What we log

| Log | What it records |
|-----|-----------------|
| **Run Log** | One entry per generated asset: the request, rule versions, model version, settings (temperature, seed), the model's raw output, validator results, fixes and retries, the final instruction, and time taken |
| **Research Log** | Every internet search, what came back, and what was passed to the model |
| **Review Log** | Whether each asset was approved or rejected, and **why** (by a person or by automatic checks) |
| **Runtime Log** | Warnings and errors in the code |

The **Review Log** is the most valuable one for improving the AI. "Rejected: snow looks like white blobs" teaches the next model version what not to do.

## Format: JSON Lines (Suggestion)

One JSON object per line. Easy to write fast, easy to search, and easy to turn into training data.

```
{"t":"2026-10-06T10:30:00.120","asset":"tree-7f3a9c21","step":"generate","model":"instructor-v0.1","temp":0.8,"seed":918273,"ms":2140}
{"t":"2026-10-06T10:30:02.300","asset":"tree-7f3a9c21","step":"validate","result":"fixed","fix":"leaves.color lightness 0.66 → 0.60"}
{"t":"2026-10-06T11:05:10.000","asset":"tree-7f3a9c21","step":"review","result":"approved","by":"auto"}
```

## Keeping logging fast

Generation is offline, so logging will not slow the game. These habits still keep it cheap:

1. **Buffer**: collect lines in memory and write them in batches.
2. **Background thread**: a separate thread writes to disk, so generation never waits for it.
3. **Log levels**: `ERROR`, `WARN`, `INFO`, `DEBUG`. Detailed logs can be switched off when you don't need them.
4. **Compile-time switch**: `DEBUG` logging can be removed from release builds completely, so it costs nothing.