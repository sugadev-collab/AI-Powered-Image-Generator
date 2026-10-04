# 03 — Request (input) and Generation Instruction (output)

## The Request: what the game sends

A request only mentions what is **different from the defaults**. You do not repeat the whole rule file.

```json
{
  "request_id": "forest-0001",
  "type": "tree",
  "seed": 12345,
  "set": {
    "leaves.color": "auto",
    "trunk.texture": "knotted"
  },
  "animation": "idle_sway",
  "at_time_ms": 0
}
```

- `type`: which Sprite Type file to use. This is always required.
- `seed`: controls random choices. If the game does not send one, the AI makes one and logs it.
- `set`: the properties to change. `part.property` names a property of one part.
- `"auto"`: the AI chooses, inside the allowed limits.
- Anything not in `set` uses the default from the Sprite Type file.

## Who wins when values disagree (precedence)

From strongest to weakest:

1. **Locked rule** in the Sprite Type file. The request cannot change it.
2. **Request value** (a strict value you set).
3. **"auto"** in the request: the AI chooses inside the allowed range.
4. **Sprite Type default**.
5. **Theme default**.

If a request tries to change a locked property, the AI ignores the change, keeps the rule, and writes a **warning** to the log. It does not crash. This keeps the game running and shows you the problem.

## The Generation Instruction: what goes to the Image Generator

This is the main output of Scope 1. Every value is final. There is no `auto` left, so the Image Generator does not have to make any decisions.

```json
{
  "request_id": "forest-0001",
  "type": "tree",
  "type_version": 1,
  "theme_version": 1,
  "seed": 12345,

  "canvas": { "w": 32, "h": 48 },
  "style": { "style": "pixel_art", "outline": { "color": "#1A1A1A", "width_px": 1 }, "light_direction": "top_left", "shading_steps": 3 },

  "parts": [
    { "id": "trunk",  "layer": 0, "region": { "x": 12, "y": 24, "w": 8,  "h": 24 }, "color": "#6B4423", "texture": "knotted" },
    { "id": "leaves", "layer": 1, "region": { "x": 2,  "y": 0,  "w": 28, "h": 28 }, "color": "#4E9A3A", "shape": "round" }
  ],

  "animation_plan": { "see": "04-animation-planning.md" }
}
```

Why it is shaped like this:
- **Versions and seed** are included so any output can be re-created and explained later.
- **Parts and layers** give the Image Generator a clear drawing order.
- **No `auto` left**: the output is fully decided. This is why we call it an *instruction*.

## Where each value came from

For every value, the Decision Log records its **source**: `locked`, `request`, `auto`, `type_default`, or `theme_default`. For `auto` values it also records the reason. See `06-logging.md`. This is how you can look back and see why a tree came out a certain way.

## Errors

If a request cannot be handled (for example, an unknown `type`), the AI Instructor returns an error object with a clear message and logs it. It never returns half an instruction.

```json
{ "request_id": "forest-0002", "error": { "code": "UNKNOWN_TYPE", "message": "No Sprite Type file for 'treee'" } }
```