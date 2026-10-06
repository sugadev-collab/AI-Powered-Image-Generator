# 04 — Animation Planning (done now, run in Scope 3)

You asked how to implement the planning part. Here is the idea, step by step.

## What "planning an animation" means

An animation is a list of small changes over time. To plan it, the AI writes down, **before the image exists**:
1. **Which parts move** (e.g. the leaves, not the trunk).
2. **Around which point** they move (the **pivot**, e.g. the top of the trunk).
3. **What changes** (rotation, position, scale, color, frame swap).
4. **When** each change happens (time in milliseconds).
5. **How many frames** and how fast (frames per second, **fps**).
6. Whether it **loops**.

This is why the Sprite Type file splits a sprite into **parts** with **pivots**. The Image Generator draws each part on its own layer, and Scope 3 later moves the layers.

## Who does what: AI for ideas, C++ for math

Language models are creative but bad at exact math. So we split the work:
- **The AI chooses the creative values**: which animation, which parts move, how big the movement is, how many frames, the style of motion.
- **C++ computes the exact numbers**: keyframe times and values, using the formulas below.
- **The validator** checks that everything stays inside the limits in the Sprite Type file.

## Keyframes

A **keyframe** is "at this time, this part is like this". Scope 3 fills in the frames between keyframes.

What the AI writes:

```json
{ "name": "idle_sway", "parts": ["leaves"], "pivot": "trunk_top", "angle_deg": 3, "frame_count": 6, "easing": "sine" }
```

What C++ turns it into:

```json
{
  "name": "idle_sway",
  "fps": 8,
  "frame_count": 6,
  "duration_ms": 750,
  "loop": true,
  "tracks": [
    {
      "part": "leaves",
      "pivot": "trunk_top",
      "property": "rotate_deg",
      "easing": "sine",
      "keyframes": [
        { "t_ms": 0,   "value": 0 },
        { "t_ms": 188, "value": 3 },
        { "t_ms": 375, "value": 0 },
        { "t_ms": 563, "value": -3 },
        { "t_ms": 750, "value": 0 }
      ]
    }
  ]
}
```

## How the times are computed

1. Duration from frame count and fps:

```
Read this:
duration_ms = frame_count / fps × 1000
            = 6 / 8 × 1000 = 750 ms
```

2. Keyframes on a **sine wave**, which gives a smooth back-and-forth swing:

```
Read this: A = largest angle, T = time for one full swing, t = current time.
angle(t) = A × sin(2π × t / T)
```

`sin` goes smoothly from 0 up to 1, back to 0, down to −1, and back to 0. Multiplying by `A` turns that into an angle from −3° to +3°. `2π` is one full circle, so one full swing takes `T` milliseconds.

3. **Validate**: the angle stays within `max_angle_deg`, the frame count is inside `frames`, all times are in order, and the pivot exists.

## Filling in frames between keyframes (Scope 3, explained now)

Between two keyframes, Scope 3 uses **linear interpolation** ("lerp"):

```
Read this: (t0, v0) and (t1, v1) are two keyframes; t is a time between them.
v = v0 + (v1 − v0) × (t − t0) / (t1 − t0)
```

`(t − t0) / (t1 − t0)` is "how far along we are", from 0 to 1. At halfway, the value is halfway between `v0` and `v1`.

## Animations that change the image itself

Some animations cannot be done by moving layers. Growing from small to large, for example, changes the appearance, not only the size. For these, the plan asks the Image Generator for **several images (stages)** and tells Scope 3 how to blend between them:

```json
{
  "name": "grow",
  "stages": [
    { "stage": 0, "size": [16, 24], "t_ms": 0,     "look": "sapling, few leaves" },
    { "stage": 1, "size": [24, 36], "t_ms": 5000,  "look": "young tree, thin trunk" },
    { "stage": 2, "size": [32, 48], "t_ms": 10000, "look": "full tree" }
  ],
  "between_stages": "crossfade"
}
```

The AI decides how each stage looks. C++ checks the sizes are inside `variants.grow`. This is how the generation step and the animation step stay connected: the AI Instructor plans **which images are needed** for the animation.

## Animation recipes (Suggestion)

Keep a small set of reusable **recipes** (sway, bob, pulse, blink, grow, flicker, twinkle) with limits in the rule files. Each Sprite Type lists the recipes it allows. The AI combines and varies recipes instead of inventing motion from nothing, which keeps animations in the game's style.

## What we do NOT do in Scope 1

We do not generate animation code and we do not play animations. That is Scope 3.