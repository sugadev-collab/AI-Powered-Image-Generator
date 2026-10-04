# 06 — Logging

You want detailed logs of the AI's actions, searches and auto choices so you can improve it, and logs of runtime problems if that does not make the engine heavy. Both are possible.

## Three kinds of log

| Log | What it records | Example |
|-----|-----------------|---------|
| **Decision Log** | Every value and where it came from (`locked`, `request`, `auto`, `type_default`, `theme_default`). For `auto`: the candidates, their scores, and the pick. | `leaves.color = #4E9A3A (auto, score 0.82, 5 candidates)` |
| **Research Log** | Every internet search by the Research Tool, and what was saved | `query "jungle fern colors" → 4 colors saved` |
| **Runtime Log** | Warnings and errors in the code | `WARN locked property 'size' ignored in request forest-0003` |

## Format: JSON Lines (Suggestion)

One JSON object per line. Easy to write quickly, easy to search, and easy to load into a tool later.

```
{"t":"2026-10-04T18:30:00.120","req":"forest-0001","step":"auto","prop":"leaves.color","pick":"#4E9A3A","score":0.82,"candidates":5,"seed":12345}
{"t":"2026-10-04T18:30:00.121","req":"forest-0001","step":"validate","result":"ok"}
```

## Keeping logging fast

Writing to a file is slow compared to the AI's work. These tricks keep logging cheap:

1. **Buffer**: collect log lines in memory and write them in batches, not one at a time.
2. **Background thread**: a separate thread does the file writing, so the AI never waits for the disk.
3. **Log levels**: `ERROR`, `WARN`, `INFO`, `DEBUG`. In a release build, keep only `WARN` and `ERROR`. Detailed decision logs can be switched on when you are tuning.
4. **Compile-time switch**: `DEBUG` logging can be fully removed from release builds, so it costs nothing at all.

## Replaying a result

Because each log entry has the request, seed and rule versions, you can run the same request again and get the exact same result. This is the main way to find out why the AI made a bad choice, and to check that a rule change fixed it.