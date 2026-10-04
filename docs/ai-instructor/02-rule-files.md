# 02 — Rule Files (the AI's default knowledge)

The rule files tell the AI what is allowed. They are why the AI "does not invent things we don't want".

There are two kinds of rule file:
1. **Theme file**: one file for the whole game's look.
2. **Sprite Type files**: one file per sprite type (tree, soil, boss, creature, weapon, resource, …).

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
  research-cache/        (filled by the Research Tool, see 01-how-it-works.md)
```

All values below are **examples to show the shape**. You decide the real values.

## 1. Theme file: the shared look

```json
{
  "theme_version": 1,
  "style": "pixel_art",
  "pixel_scale": 1,
  "outline": { "enabled": true, "color": "#1A1A1A", "width_px": 1 },
  "light_direction": "top_left",
  "shading_steps": 3,
  "max_colors_per_sprite": 16,
  "min_contrast": 0.15,
  "global_palette": ["#1A1A1A", "#3A7D2C", "#6B4423", "#C9A66B", "#E8E0C8"]
}
```

Why each part matters:
- **style**: the most important rule for a consistent look. Pixel art, painted, flat vector and so on look very different.
- **outline**: if some sprites have outlines and some do not, the game looks mixed.
- **light_direction**: every sprite must be lit from the same side, or shadows will look wrong next to each other.
- **shading_steps**: how many light and dark versions of each color are used. Keeping this the same keeps the style the same.
- **max_colors_per_sprite** and **global_palette**: a limited palette is one of the strongest tools for making many sprites look like one game.
- **min_contrast**: used by the AI when it picks `auto` colors (see `01-how-it-works.md`).

## 2. Sprite Type file: example for a tree

```json
{
  "type": "tree",
  "type_version": 1,
  "category": "environment",

  "size": { "default": [32, 48], "locked": true },

  "parts": [
    {
      "id": "trunk",
      "layer": 0,
      "region": { "x": 12, "y": 24, "w": 8, "h": 24 },
      "color":   { "default": "#6B4423", "allowed": ["#5A3A1E", "#6B4423", "#7A5230"] },
      "texture": { "default": "bark_lines", "allowed": ["bark_lines", "smooth", "knotted"] }
    },
    {
      "id": "leaves",
      "layer": 1,
      "region": { "x": 2, "y": 0, "w": 28, "h": 28 },
      "color":   { "default": "#3A7D2C", "allowed_hue": [70, 150], "allowed_lightness": [0.25, 0.6] },
      "shape":   { "default": "round", "allowed": ["round", "pine", "droopy"] }
    }
  ],

  "pivots": { "trunk_top": { "x": 16, "y": 26 } },

  "animations": {
    "idle_sway": { "moving_parts": ["leaves"], "pivot": "trunk_top", "max_angle_deg": 4, "frames": [4, 8], "fps": 8, "loop": true }
  },

  "variants": {
    "grow": { "size_min": [16, 24], "size_max": [32, 48], "keep_ratio": true }
  },

  "forbidden": ["faces", "text", "glow"]
}
```

What each part means, and why the AI needs it:

| Field | Meaning | Why the AI needs it |
|-------|---------|--------------------|
| `type`, `category` | What this sprite is | The request says `type: tree`, so the AI knows which file to use. |
| `type_version` | Version number of this file | Logs record the version, so old results can be explained after you change the rules. |
| `size` + `locked` | Default size, and whether a request may change it | This is your "size is fixed" rule. |
| `parts` | The pieces of the sprite (trunk, leaves) | Lets you say *where* each color goes. "Green and brown" is not enough; "trunk is brown, leaves are green" is. |
| `layer` | Drawing order (0 is drawn first, at the back) | Leaves must be drawn on top of the trunk. |
| `region` | Where the part sits on the canvas, in pixels | Tells the Image Generator the exact location. |
| `default` | Value used when the request does not mention it | Your rule: "if we didn't mention a property, use the default". |
| `allowed` / `allowed_hue` / `allowed_lightness` | The limits for `auto` | The AI can only choose inside these. This keeps `auto` from breaking the look. |
| `pivots` | Points that parts rotate or grow around | Needed now for the Animation Plan, and later by Scope 3. |
| `animations` | Which animations this type can have, and their limits | The AI plans only animations that are allowed here. |
| `variants` | Allowed size changes, e.g. growing from small to large | Your "size can change within a limit" rule. |
| `forbidden` | Things that must never appear | A safety net against unwanted features. |

## Colors, explained for a beginner

Colors are written as `#RRGGBB`, the amount of red, green and blue (each from `00` to `FF`).

For `auto` choices it is easier to think in **HSL**:
- **Hue**: the position on the color wheel, from 0° to 360°. About 0° is red, 120° is green, and 240° is blue.
- **Saturation**: how strong the color is, from gray (0) to vivid (1).
- **Lightness**: from black (0) to white (1).

`"allowed_hue": [70, 150]` means "any color from yellow-green to blue-green". That is a natural way to say "leaves can be any green".

## Tips for writing good rule files

1. Start with **one** type (the tree) and make it work from start to finish before adding more types.
2. Prefer **allowed lists** over open ranges at first. They are easier to control.
3. Every new property needs a `default`. Then requests can stay short.
4. Lock the things that must never change in the game (often size, outline, light direction).