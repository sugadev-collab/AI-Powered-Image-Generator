# 05 — Full Sprite Generation, Stats and Rarity

There are two jobs:

| Job | Example | How much the AI decides |
|-----|---------|------------------------|
| **Decorate** an existing sprite | Change a tree's look for Christmas, add a small sway | Some values and details |
| **Create a full sprite** | A new boss or crafted weapon, with stats and rarity | Almost everything, under the rules |

Full sprite creation happens in specific areas: boss fights, weapon crafting and other crafting.

## The idea: generate → check → fix

1. **Generate**: the AI designs the whole sprite: parts, colors, details, animation and stats.
2. **Check**: the validator tests every rule, including rules that connect values (for example, "a legendary weapon must have a glow").
3. **Fix**: small problems are fixed automatically. Big problems: the AI tries again (up to a set number of tries). Every fix is logged.

Nothing is output until the check passes, so a sprite that breaks the rules never reaches the game.

## Rarity: picked by C++, designed by the AI (Suggestion)

Rarity affects game balance, so the **chance** of each rarity should be exact. Let C++ pick the rarity with weights, and then let the AI design a sprite that **fits** that rarity.

```json
{
  "rarity_version": 1,
  "tiers": [
    { "id": "common",    "weight": 60, "stat_budget": [10, 20], "visual": "plain, muted colors",           "glow": false },
    { "id": "rare",      "weight": 30, "stat_budget": [20, 35], "visual": "brighter colors, one extra detail", "glow": false },
    { "id": "legendary", "weight": 10, "stat_budget": [35, 50], "visual": "vivid, ornate, two extra details",  "glow": true }
  ]
}
```

- **weight**: how often each tier is picked. Here common is picked 60 times out of 100.
- **stat_budget**: the total stat points a sprite of this tier can have.
- **visual**: how rarity shows on the sprite, as text the AI reads. Players should see rarity at a glance.

```
Read this: add up all weights (60 + 30 + 10 = 100).
Pick a random number r from 0 to 99.
r < 60          → common
60 ≤ r < 90     → rare
90 ≤ r < 100    → legendary
```

## Stats: proposed by the AI, balanced by C++

The AI suggests stats that **match its design**. For example, a heavily armored boss gets more defense. C++ then makes sure the total equals the budget and each stat is inside its limits:

```
Read this:
B      = the stat budget (inside the tier's range)
sum    = stat_1 + stat_2 + ... + stat_n    (what the AI suggested)
stat_i = stat_i × B / sum                  ← scale every stat so they add up to B
then clamp each stat_i to its [min_i, max_i]
```

Scaling keeps the AI's idea ("lots of defense, little speed") but makes the total fair. Clamping means "if it is above the maximum, set it to the maximum".

## Linking stats to looks (Suggestion)

Ask the AI (through the rule file descriptions) to make visuals follow the stats. For example:
- higher `attack` → a larger blade, within the allowed size range
- higher `defense` → thicker armor parts
- `element: fire` → hues limited to the red and orange range (this one is a hard limit the validator checks)

Then a sprite's look tells the player something true about it.

## Making every boss different: the variety check

To make sure a new boss is not too close to one that already exists, compare their key choices (parts, shapes, main colors, details):

```
Read this: A and B are the sets of key choices of two bosses.
similarity = (choices both share) / (all different choices in A and B together)
```

This gives 0 (nothing in common) to 1 (identical). If the similarity to any existing boss is above a limit (for example 0.7), the AI tries again. The limit goes in the rule file so you can tune it.

## Output

A full sprite's output is a normal Generation Instruction (see `03-request-and-output.md`) plus a `rarity` value and a `stats` block:

```json
{ "rarity": "rare", "stats": { "attack": 18, "defense": 9 }, "parts": [ "..." ], "animation_plan": { "...": "..." } }
```