# 03 — Request (input) and Generation Instruction (output)

## The Request: what asks for assets

A request only mentions what is **different from the defaults**. You never repeat the rule file.

```json
{
  "request_id": "xmas-trees-001",
  "type": "tree",
  "event": "christmas",
  "count": 200,
  "set": {
    "size": [1, 2],
    "leaves.color": "auto",
    "trunk.texture": "knotted"
  },
  "animation": "auto",
  "release_at": "2026-12-20T00:00:00"
}
```

- `type`: which Sprite Type file to use. Always required.
- `event`: optional. Which Event file to apply.
- `count`: how many different assets to make. Each one is generated separately, so each one is different.
- `set`: properties to change. `part.property` names a property of one part.
- `size`: in tiles. An exact size like `[1, 2]` is followed strictly. `"auto"` lets the AI pick inside the type's `min` and `max`.
- `"auto"`: the AI chooses inside the limits.
- `release_at`: when the assets should appear in the game (see `01-how-it-works.md`).
- Anything not in `set` uses the default from the rule files.

## Who wins when values disagree (precedence)

From strongest to weakest:

1. **Locked** in the Sprite Type file. Nothing can change it, including events.
2. **Request value** (a strict value a developer set).
3. **"auto"**: the AI chooses inside the limits. The limits are the Sprite Type limits, as changed by the Event file.
4. **Event default**.
5. **Sprite Type default** (which can itself be `"auto"`).
6. **Theme default**.

If a request tries to change a locked property, the change is ignored, the rule is kept, and a **warning** is logged. Nothing crashes.

## The Generation Instruction: what goes to the Image Generator

This is the main output of Scope 1. Every value is final, with no `auto` left, so the Image Generator does not have to make any decisions.

```json
{
  "asset_id": "tree-7f3a9c21",
  "request_id": "xmas-trees-001",
  "type": "tree",
  "event": "christmas",
  "versions": { "type": 1, "theme": 1, "event": 1, "model": "gemma-4-e4b-it + instructor-lora-v0.1" },
  "seed": 918273,
  "release_at": "2026-12-20T00:00:00",
  "status": "pending_review",

  "canvas": { "tiles": [1, 2], "w": 32, "h": 64 },
  "style": { "style": "pixel_art", "outline": { "color": "#1A1A1A", "width_px": 1 }, "light_direction": "top_left", "shading_steps": 3 },

  "parts": [
    { "id": "trunk",  "layer": 0, "region": { "x": 0.375, "y": 0.5, "w": 0.25, "h": 0.5  }, "color": "#6B4423", "texture": "knotted" },
    { "id": "leaves", "layer": 1, "region": { "x": 0.06,  "y": 0.0, "w": 0.88, "h": 0.58 }, "color": "#4E9A3A", "shape": "pine" }
  ],
  "details": [
    { "id": "snow_cap", "on": "leaves", "amount": 0.4 },
    { "id": "string_lights", "on": "leaves", "colors": ["#D7263D", "#F2C14E"] }
  ],

  "animation_plan": { "see": "04-animation-planning.md" }
}
```

Why it is shaped like this:
- **asset_id**: a unique name for this asset. It is also what an NFT would need later.
- **versions and seed**: recorded so any asset can be explained or recreated when debugging.
- **status**: `pending_review` → `approved` or `rejected`. Only approved assets are exported to the game.
- **canvas**: the size in tiles and in pixels, so the Image Generator draws exactly the size the game needs.
- **parts and layers**: a clear drawing order for the Image Generator.
- **No `auto` left**: the output is fully decided. That is why it is an *instruction*.

The model's **thinking** is not part of the instruction. It is saved in the Run Log (see `06-logging.md`).

## How results reach the game: the Asset Package

The Image Creator Tool runs separately from the game. When assets are approved, it **exports** them into an Asset Package, a folder the game reads:

```
asset-package/
  manifest.json             (every asset: id, type, file names, release_at)
  tree-7f3a9c21/
    image.png               (from Scope 2)
    animation.json          (from Scope 3)
    info.json               (the final instruction: metadata, stats, rarity)
```

```json
{
  "package_version": 1,
  "assets": [
    { "asset_id": "tree-7f3a9c21", "type": "tree", "release_at": "2026-12-20T00:00:00", "folder": "tree-7f3a9c21" }
  ]
}
```

The game never talks to the AI. It only reads the manifest and loads each asset when its `release_at` arrives.

## Errors

If a request cannot be handled, the AI Instructor returns an error object with a clear message and logs it. It never returns half an instruction.

```json
{ "request_id": "xmas-trees-002", "error": { "code": "UNKNOWN_TYPE", "message": "No Sprite Type file for 'treee'" } }
{ "request_id": "xmas-trees-003", "error": { "code": "VALIDATION_FAILED", "message": "3 tries; leaves.color kept breaking allowed_hue" } }
{ "request_id": "xmas-trees-004", "error": { "code": "SIZE_OUT_OF_LIMITS", "message": "size [4, 4] is bigger than tree max [2, 3]" } }
```