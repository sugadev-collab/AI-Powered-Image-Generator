# 01 — How the AI Instructor Works

## The steps (the "pipeline")

A pipeline is a list of steps where the output of one step is the input of the next.

```
Request from game
      │
      ▼
1. Load rules ........ Theme file + Sprite Type file (loaded once at startup, kept in memory)
      │
      ▼
2. Merge ............. Request values + defaults. Anything not mentioned uses the default.
      │
      ▼
3. Check locks ....... Locked properties (e.g. a fixed tree size) cannot be changed by the request.
      │
      ▼
4. Resolve "auto" .... For every "auto" value, the AI picks a value inside the allowed range
      │                 that fits the strict values around it.
      ▼
5. Validate .......... Check the result follows every rule. If something breaks a rule, fix it
      │                 (for example, clamp a size back into its limit) and log the fix.
      ▼
6. Plan animation .... Choose the animation, which parts move, the pivots, the timing.
      │
      ▼
7. Output ............ Generation Instruction + Animation Plan (+ Decision Log entries)
```

Every step is plain, fast C++. Most requests should take far less than a millisecond.

## My opinion: what kind of "AI" this should be

You said the AI does not need to understand full text like a chat model. I agree, and I think this is a strength.

Your requests are **structured**: `type = tree`, `size = 32x48`, `color = auto`. For structured input, the right tool is a **rule-based decision engine with seeded choices and scoring**. This is the same kind of AI used in many games for decisions (often called "game AI" or "procedural generation").

Why this is a good fit (Suggestion):
- **Fast**: no neural network runs, so it is suitable for an engine used in many places at the same time.
- **Free**: no training and no GPU needed. This matches your 0 USD budget.
- **Follows rules exactly**: the AI cannot invent things you did not allow, because it only chooses from what the rule files allow.
- **Repeatable**: with a seed, the same request always gives the same result. This makes the logs useful for improving the AI.
- **Tunable with JSON**: changing the AI's behaviour means editing a rule file, not retraining a model.

A language model (LLM) can be added **later** as an optional extra, for example to turn free text ("make a scary swamp tree") into a structured request. It should not sit in the fast path.

## How the AI makes a good "auto" choice

When a value is `auto`, the AI does this:
1. Make a list of **candidates** that are allowed by the rules.
2. Give each candidate a **score** for how well it fits the strict values already decided.
3. Pick the best one, or a random one from the top few, using the seed. This keeps results varied but always good.

Example: the leaves color is `auto` and the trunk is a fixed brown. The AI prefers leaf colors that have enough contrast with the trunk, so the two parts do not blend together.

A simple contrast rule:

```
Read this: L is "lightness", from 0 (black) to 1 (white).
contrast_ok = |L_leaves − L_trunk| ≥ min_contrast
```

`|x|` means "the size of x, ignoring minus signs". If the leaves and trunk are almost equally light, they look like one blob. The rule rejects that. `min_contrast` comes from the Theme file, so you can tune it.

## Internet access (your idea)

You want the AI to learn from the internet when choosing `auto` values.

**Suggestion:** do not search the internet during a game request. It is slow, it fails when offline, and results change from day to day, so the same request would give different sprites.

Instead, make a separate **Research Tool** that you run when you want:
1. It searches the internet (for example, for real colors of jungle plants).
2. It saves what it found into a **cache file** next to the rules.
3. It logs every search and result, so you can review them.
4. The AI Instructor then uses the cache file like any other rule file.

You get the benefit of the internet without making the engine slow or unpredictable.

## Time ordering (your idea)

You want to put things in time order and have the AI create them in that order.

This is cheap and does not need to be inside the AI's decision logic:
- Each request can have an `at_time_ms` field (time in milliseconds).
- Requests wait in a **priority queue**, which is a list that always gives you the earliest item first.
- Inside an Animation Plan, every keyframe also has a time (`t_ms`). See `04-animation-planning.md`.

Adding to a priority queue costs roughly `log2(n)` steps for `n` items. With 1,000 items that is about 10 steps, so it will not hurt performance.

## Used in many places at the same time

To let many parts of the game use the AI Instructor at the same time safely:
- Rule files are loaded **once** and are **read-only** after that.
- Each request carries its own seed and its own working data.

Because nothing shared is changed, many threads can run requests at the same time without locks. A lock is a "wait your turn" mechanism, and it slows things down.