# 05 — Full Sprite Generation, Stats and Rarity

There are two jobs, and they use the AI differently:

| Job | Example | How much the AI decides |
|-----|---------|------------------------|
| **Decorate** an existing sprite | Change a tree's leaf color, add a small sway | Little: a few `auto` values |
| **Create a full sprite** | A new boss, a new crafted weapon, with stats and rarity | A lot: many parts, stats and rarity, all under the rules |

Full sprite creation happens in specific areas: boss fights, weapon crafting and other crafting.

## The idea: generate → check → fix

For a full sprite, the AI follows the same pipeline as in `01-how-it-works.md`, but most values are `auto`. To keep it under the rules:

1. **Generate**: pick every value from inside its allowed range.
2. **Check**: test all rules, including rules that connect values (for example, "a legendary weapon must have a glow").
3. **Fix**: if a rule fails, change the value to the nearest allowed one, or pick again (up to a set number of tries). Log every fix.

The AI can never output something that breaks the rules, because nothing is returned until the check passes.

## Rarity file (example shape)

```json
{
  "rarity_version": 1,
  "tiers": [
    { "id": "common",    "weight": 60, "stat_budget": [10, 20], "visual": { "saturation_max": 0.5, "extra_details": 0, "glow": false } },
    { "id": "rare",      "weight": 30, "stat_budget": [20, 35], "visual": { "saturation_max": 0.7, "extra_details": 1, "glow": false } },
    { "id": "legendary", "weight": 10, "stat_budget": [35, 50], "visual": { "saturation_max": 0.9, "extra_details": 2, "glow": true } }
  ]
}
```

- **weight**: how often each tier is picked when rarity is `auto`. Here common is chosen 60 times out of 100.
- **stat_budget**: the total stat points a sprite of this tier can have.
- **visual**: how rarity shows on the sprite. Players should be able to see rarity at a glance.

## Picking a rarity with weights

```
Read this: add up all weights (60 + 30 + 10 = 100).
Pick a random number r from 0 to 99 using the seed.
r < 60          → common
60 ≤ r < 90     → rare
90 ≤ r < 100    → legendary
```

## Sharing stat points (the stat budget)

Each stat-carrying type (boss, creature, weapon) lists its stats with a minimum and maximum, for example `attack [1, 30]` and `defense [1, 30]`.

```
Read this:
1. B = stat budget, chosen inside the tier's range using the seed.
2. For each stat i, pick a random weight r_i between 0 and 1.
3. share_i = r_i / (r_1 + r_2 + ... + r_n)       ← all shares add up to 1
4. stat_i  = min_i + share_i × B, then clamp to max_i
```

The shares split the budget like slices of a pie. Clamping means "if it is above the maximum, set it to the maximum".

Sprite Type files can add **bias** so types feel different. For example, a boss type can say attack gets at least 40% of the budget.

## Linking stats to looks (Suggestion)

Make the visuals follow the stats through simple rules in the Sprite Type file. For example:
- higher `attack` → a larger blade, within the allowed size range
- higher `defense` → thicker armor parts
- an `element: fire` value → hues limited to the red and orange range

Then a sprite's look tells the player something true about it.

## Output

A full sprite's output is a normal Generation Instruction (see `03-request-and-output.md`) plus a `stats` block and a `rarity` value:

```json
{ "rarity": "rare", "stats": { "attack": 18, "defense": 9 }, "parts": [ "..." ], "animation_plan": { "...": "..." } }
```