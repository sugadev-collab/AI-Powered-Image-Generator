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

## Keyframes

A **keyframe** is "at this time, this part is like this". Scope 3 fills in the frames between keyframes.

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

## How the AI creates this plan

1. Read the requested animation (`idle_sway`) from the Sprite Type file's `animations` list. If the animation is not allowed for this type, log a warning and skip it.
2. Choose free values inside their limits using the seed. Here, angle = 3° (the limit is 4°) and 6 frames (the limit is 4 to 8).
3. Work out the times from fps and frame count:

```
Read this:
duration_ms = frame_count / fps × 1000
            = 6 / 8 × 1000 = 750 ms
```

4. Put keyframes on a **sine wave**, which gives a smooth back-and-forth swing:

```
Read this: A = largest angle, T = time for one full swing, t = current time.
angle(t) = A × sin(2π × t / T)
```

`sin` goes smoothly from 0 up to 1, back to 0, down to −1, and back to 0. Multiplying by `A` turns that into an angle from −3° to +3°. `2π` is one full circle, so one full swing takes `T` milliseconds.

5. **Validate**: the angle stays within the limit, all times are in order, and the pivot exists.

## Filling in frames between keyframes (Scope 3, explained now)

Between two keyframes, Scope 3 uses **linear interpolation** ("lerp"):

```
Read this: (t0, v0) and (t1, v1) are two keyframes; t is a time between them.
v = v0 + (v1 − v0) × (t − t0) / (t1 − t0)
```

`(t − t0) / (t1 − t0)` is "how far along we are", from 0 to 1. At halfway the value is halfway between `v0` and `v1`.

## Animations that change the image itself

Some animations cannot be done by moving layers. Growing from small to large, for example, changes the appearance, not only the size. For these, the plan asks the Image Generator for **several images (stages)** and tells Scope 3 how to blend between them:

```json
{
  "name": "grow",
  "stages": [
    { "stage": 0, "size": [16, 24], "t_ms": 0 },
    { "stage": 1, "size": [24, 36], "t_ms": 5000 },
    { "stage": 2, "size": [32, 48], "t_ms": 10000 }
  ],
  "between_stages": "crossfade"
}
```

So the AI Instructor plans **which images are needed** for the animation. That is how the generation step and the animation step stay connected.

## Animation recipes (Suggestion)

Keep a small set of reusable **recipes** (sway, bob, pulse, blink, grow, flicker) with limits in the rule files. Each Sprite Type lists the recipes it allows. The AI combines recipes instead of inventing motion. This keeps animations consistent with the game's style.

## What we do NOT do in Scope 1

We do not generate animation code and we do not play animations. That is Scope 3. Scope 1 only outputs the plan above.