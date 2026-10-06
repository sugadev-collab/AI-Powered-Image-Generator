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
- `"auto"`: the AI chooses inside the limits.
- `release_at`: when the assets should appear in the game (see `01-how-it-works.md`).
- Anything not in `set` uses the default from the rule files.

## Who wins when values disagree (precedence)

From strongest to weakest:

1. **Locked** in the Sprite Type file. Nothing can change it, including events.
2. **Request value** (a strict value you set).
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
  "versions": { "type": 1, "theme": 1, "event": 1, "model": "instructor-v0.1" },
  "seed": 918273,
  "release_at": "2026-12-20T00:00:00",
  "status": "pending_review",

  "canvas": { "w": 32, "h": 48 },
  "style": { "style": "pixel_art", "outline": { "color": "#1A1A1A", "width_px": 1 }, "light_direction": "top_left", "shading_steps": 3 },

  "parts": [
    { "id": "trunk",  "layer": 0, "region": { "x": 12, "y": 24, "w": 8,  "h": 24 }, "color": "#6B4423", "texture": "knotted" },
    { "id": "leaves", "layer": 1, "region": { "x": 2,  "y": 0,  "w": 28, "h": 28 }, "color": "#4E9A3A", "shape": "pine" }
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
- **status**: `pending_review` → `approved` or `rejected`. Only approved assets reach the game.
- **parts and layers**: a clear drawing order for the Image Generator.
- **No `auto` left**: the output is fully decided. That is why it is an *instruction*.

## Errors

If a request cannot be handled, the AI Instructor returns an error object with a clear message and logs it. It never returns half an instruction.

```json
{ "request_id": "xmas-trees-002", "error": { "code": "UNKNOWN_TYPE", "message": "No Sprite Type file for 'treee'" } }
{ "request_id": "xmas-trees-003", "error": { "code": "VALIDATION_FAILED", "message": "3 tries; leaves.color kept breaking allowed_hue" } }
```