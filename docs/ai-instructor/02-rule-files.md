# 02 — Rule Files (the AI's guardrails)

The rule files tell the AI **what it must keep and how far it may go**. They do not make the AI's choices. They stop it from inventing things we don't want.

There are three kinds of rule file:
1. **Theme file**: one file for the whole game's look.
2. **Sprite Type files**: one per sprite type (tree, soil, boss, creature, weapon, resource, …).
3. **Event files**: one per game event (Christmas, Halloween, …). Optional.

Suggested folder layout (to create when we start coding):

```
rules/
  theme.json
  rarity.json            (see 05-full-sprites-stats-rarity.md)
  types/
    tree.json
    soil.json
    boss.json
    ...
  events/
    christmas.json
  research-cache/        (filled by the research step, see 01-how-it-works.md)
```

All values below are **examples to show the shape**. You decide the real values.

## Why the rule files have text descriptions

Our AI is a thinking language model, so it understands text. A one-line `description` such as "leaves are soft round clusters, no sharp edges" helps it match the style much better than numbers alone. Numbers set the hard limits, and text describes the feeling.

## 1. Theme file: the shared look

```json
{
  "theme_version": 1,
  "description": "Cozy jungle pixel art. Soft shapes, warm light, readable silhouettes.",
  "style": "pixel_art",
  "tile_size_px": 32,
  "outline": { "enabled": true, "color": "#1A1A1A", "width_px": 1 },
  "light_direction": "top_left",
  "shading_steps": 3,
  "max_colors_per_sprite": 16,
  "min_contrast": 0.15,
  "global_palette": ["#1A1A1A", "#3A7D2C", "#6B4423", "#C9A66B", "#E8E0C8"]
}
```

Why each part matters:
- **style** and **description**: the most important rules for a consistent look.
- **tile_size_px**: the size of one tile in the game grid. Every sprite size is a whole number of tiles, so every sprite fits the grid.
- **outline**: if some sprites have outlines and some do not, the game looks mixed.
- **light_direction**: every sprite must be lit from the same side, or shadows look wrong next to each other.
- **shading_steps**: how many light and dark versions of each color are used. Keeping this the same keeps the style the same.
- **max_colors_per_sprite** and **global_palette**: a limited palette is one of the strongest tools for making many sprites look like one game.
- **min_contrast**: neighbouring parts must differ enough in lightness, or they blend into one blob. The validator checks this:

```
Read this: L is "lightness", from 0 (black) to 1 (white).
contrast_ok = |L_part_a − L_part_b| ≥ min_contrast
```
`|x|` means "the size of x, ignoring minus signs".

## 2. Sprite Type file: example for a tree

```json
{
  "type": "tree",
  "type_version": 1,
  "category": "environment",
  "description": "A jungle tree. Thick trunk, rounded leafy crown.",

  "ai": { "temperature": 0.8, "thinking_budget": "medium" },

  "size": { "unit": "tiles", "default": [1, 2], "min": [1, 1], "max": [2, 3] },

  "parts": [
    {
      "id": "trunk",
      "layer": 0,
      "region": { "x": 0.375, "y": 0.5, "w": 0.25, "h": 0.5 },
      "color":   { "default": "#6B4423", "allowed": ["#5A3A1E", "#6B4423", "#7A5230"] },
      "texture": { "default": "auto", "allowed": ["bark_lines", "smooth", "knotted"] }
    },
    {
      "id": "leaves",
      "layer": 1,
      "region": { "x": 0.06, "y": 0.0, "w": 0.88, "h": 0.58 },
      "color":   { "default": "auto", "allowed_hue": [70, 150], "allowed_lightness": [0.25, 0.6] },
      "shape":   { "default": "round", "allowed": ["round", "pine", "droopy"] }
    }
  ],

  "details": { "allowed": ["moss", "mushrooms", "bird_nest", "cracks"], "max": 2 },

  "pivots": { "trunk_top": { "x": 0.5, "y": 0.54 } },

  "animations": {
    "idle_sway": { "moving_parts": ["leaves"], "pivot": "trunk_top", "max_angle_deg": 4, "frames": [4, 8], "fps": 8, "loop": true }
  },

  "variants": {
    "grow": { "allowed": true, "stages": [2, 5] }
  },

  "forbidden": ["faces", "text", "glow"]
}
```

What each part means, and why the AI needs it:

| Field | Meaning | Why it matters |
|-------|---------|----------------|
| `type`, `category`, `description` | What this sprite is | The request says `type: tree`, so the right file is used. The text helps the model understand it. |
| `type_version` | Version number of this file | Logs record it, so old results can be explained after you change the rules. |
| `ai.temperature` | How adventurous the AI is for this type | Bosses can be wild and soil tiles calm. See `01-how-it-works.md`. |
| `ai.thinking_budget` | How much the model may think for this type | More thinking gives better decisions but takes longer. |
| `size` | Default size in **tiles**, plus the smallest and largest size allowed | **Developers decide each sprite's size** so it fits the tile grid. A request can set an exact size (followed strictly) or `auto` (the AI picks between `min` and `max`). If a type must *always* keep one size, add `"locked": true`. |
| `parts` | The pieces of the sprite (trunk, leaves) | Lets you say **where** each color goes. "Trunk is brown, leaves are green" is much clearer than "green and brown". |
| `layer` | Drawing order (0 is at the back) | Leaves must be drawn on top of the trunk. Separate layers also let Scope 3 move parts. |
| `region` | Where the part sits, as **fractions from 0 to 1** of the canvas | Fractions still work when the developer picks a different size. `x: 0.5` always means "the middle". |
| `default` | Used when the request does not mention the property | Your rule: "if we didn't mention a property, use the default". A default of `"auto"` means "dynamic unless the request says otherwise". |
| `allowed` / `allowed_hue` / `allowed_lightness` | The limits for the AI | The AI is free **inside** these limits. |
| `details` | Small extras the AI may add, and how many | Variety without inventing unwanted things. |
| `pivots` | Points that parts rotate or grow around (also fractions) | Needed for the Animation Plan. |
| `animations` | Which animations this type can have, and their limits | The AI plans only these. |
| `variants.grow` | Whether this type can grow, and in how many stages | Your "size can change within a limit" rule. Every stage size stays inside `size.min` and `size.max`. |
| `forbidden` | Things that must never appear | A last safety net, checked by the validator. |

From tiles to pixels:

```
Read this: canvas size in pixels = size in tiles × tile_size_px
1×2 tiles with 32 px tiles → 32 × 64 pixels
```

## 3. Event file: example for Christmas

```json
{
  "event": "christmas",
  "event_version": 1,
  "description": "Winter holiday. Snow covers top surfaces. Warm festive lights.",
  "applies_to": ["tree", "soil"],
  "palette_add": ["#FFFFFF", "#D7263D", "#F2C14E"],
  "details_add": { "tree": ["snow_cap", "string_lights", "ornaments"], "soil": ["snow_patch"] },
  "limits_change": { "tree": { "leaves.color.allowed_lightness": [0.25, 0.75] } },
  "animations_add": { "tree": ["lights_twinkle"] }
}
```

An event can **add** options and **change limits**. It can **never** override a strict value from the request or a locked value in the type file. So during Christmas, the trees get snow and lights, but their size stays whatever the developer set.

## Colors, explained for a beginner

Colors are written as `#RRGGBB`, the amount of red, green and blue (each from `00` to `FF`).

For limits it is easier to think in **HSL**:
- **Hue**: the position on the color wheel, from 0° to 360°. About 0° is red, 120° is green, and 240° is blue.
- **Saturation**: how strong the color is, from gray (0) to vivid (1).
- **Lightness**: from black (0) to white (1).

`"allowed_hue": [70, 150]` means "any color from yellow-green to blue-green", a natural way to say "leaves can be any green".

## Tips for writing good rule files

1. Start with **one** type (the tree) and one event, and make them work from start to finish first.
2. Every property needs a `default` (a value, or `"auto"`). Then requests can stay short.
3. Lock only what must never change for that type (often outline and light direction). Leave size to developers per request unless a type must always fill the same tiles.
4. Keep descriptions short and concrete. "Rounded crown, no sharp edges" is better than "beautiful tree".