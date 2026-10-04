# 08 — Roadmap (build order)

Small steps. Each step ends with something you can run and test. We build for **one** Sprite Type (tree) first, then add more.

| Step | Goal | Done when |
|------|------|-----------|
| **M0** | Project setup: CMake, folder layout, doctest, our small logger | `build` runs and one test passes |
| **M1** | Load `theme.json` and `types/tree.json` into C++ structures | A test prints the loaded tree rules |
| **M2** | Read a Request, merge it with defaults, and enforce locks | A request with no `set` gives the default tree; changing a locked size is ignored and logged |
| **M3** | Seeded random + `auto` resolution with scoring | The same seed always gives the same color; a different seed gives a different color, always inside the limits |
| **M4** | Validate and fix | A rule-breaking value is fixed and logged |
| **M5** | Output the Generation Instruction (JSON) | The output matches the format in `03-request-and-output.md` |
| **M6** | Animation Plan (keyframes, stages) | The `idle_sway` and `grow` plans match `04-animation-planning.md` |
| **M7** | Decision Log + Runtime Log (buffered, background thread) | Logs are written and a request can be replayed from its log |
| **M8** | Time ordering (`at_time_ms` + priority queue) | Requests come out in time order |
| **M9** | Rarity + stats + full sprite generation (start with one weapon type) | Generated stats always stay inside the budget and limits |
| **M10** | More Sprite Types: soil, creature, boss, crafting item, resource | Each new type works by only adding a JSON file |
| **M11** | Research Tool (optional): internet search → cache file → log | Cached results can be used by `auto` |

M10 is the real test of the design: **a new sprite type should need only a new rule file, with no new C++ code.**

## Suggested code layout (to create at M0)

```
docs/ai-instructor/     (these documents)
rules/                  (theme, rarity, types/)
src/instructor/         (the AI Instructor code)
tests/                  (doctest tests)
CMakeLists.txt
```