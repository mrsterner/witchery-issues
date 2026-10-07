<div align="center">
  <img src="logo.png" alt="Ritual Logo" width="256"/>
</div>

# Just Another Witchery Remake

[![CurseForge](https://img.shields.io/badge/Download%20on-CurseForge-orange?style=flat-square)](https://legacy.curseforge.com/minecraft/mc-mods/just-another-witchery-remake)
[![Modrinth](https://img.shields.io/badge/Download%20on-Modrinth-green?style=flat-square)](https://modrinth.com/mod/just-another-witchery-remake)

## Description

- JEI support
- Neoforge support only as of 0.4+
- 1.21.1 and 26.1.2
- Uses Modomomicon for in-game wiki

## JSON Structures
<details>
<summary>Ritual JSON, directory: ritual</summary>

Rituals are data-driven and run through an ordered list of `ritual_events`. Each event is either a **requirement** (at least one has to be present), an **output** (a produced effect), or **neutral** (an ambient/ongoing effect), and all requirements need to come before anything else in the list. As soon as the first output event is reached, the ritual enters its "output phase": `altar_power` gets consumed, `altar_power_per_second` starts draining, and a sound plays. Outputs can also run in `parallel`.

```json
{
  "type": "witchery:ritual",
  "altar_power": 2000,
  "altar_power_per_second": 0,
  "block_mapping": {
    "G": "witchery:golden_chalk",
    "M": "witchery:otherwhere_chalk",
    "S": "witchery:otherwhere_chalk"
  },
  "celestial_conditions": ["night"],
  "weather": ["storm"],
  "require_cat": false,
  "coven_count": 0,
  "pattern": [
    "___MMMMM___",
    "__M_____M__",
    "_M__SSS__M_",
    "M__S___S__M",
    "M_S_____S_M",
    "M_S__G__S_M",
    "M_S_____S_M",
    "M__S___S__M",
    "_M__SSS__M_",
    "__M_____M__",
    "___MMMMM___"
  ],
  "ritual_events": [
    {
      "type": "witchery:consume_items",
      "items": [
        { "count": 1, "item": "witchery:spirit_of_otherwhere" },
        { "count": 1, "tag": "minecraft:logs" }
      ],
      "timeout": 400
    },
    {
      "type": "witchery:consume_sacrifice",
      "entities": [
        "minecraft:pig",
        ["minecraft:villager", "minecraft:cow"],
        "#minecraft:raiders",
        ["#c:animals", "minecraft:villager"]
      ],
      "timeout": 600,
      "drop": false
    },
    {
      "type": "witchery:parallel",
      "events": [
        {
          "type": "witchery:run_command",
          "command": "witchery infusion setAndKill {owner} otherwhere",
          "min_witch_power": 2,
          "max_witch_power": 3
        },
        {
          "type": "witchery:summon_item",
          "items": [{ "count": 1, "id": "minecraft:stick" }],
          "floating": false
        }
      ]
    }
  ]
}
```

### Fields

| Key                     | Description                                                                                                                |
|--------------------------|------------------------------------------------------------------------------------------------------------------------------|
| `type`                  | Always `"witchery:ritual"`.                                                                                                  |
| `ritual_events`         | Ordered list of ritual events (see below). Requirement events must all precede non-requirement events.                      |
| `pattern`               | Visual layout using characters defined in `block_mapping`, forming the ritual circle shape.                                 |
| `block_mapping`         | Mapping of pattern characters to block IDs.                                                                                  |
| `altar_power`           | Amount of altar power consumed once the ritual enters its output phase.                                                     |
| `altar_power_per_second`| Amount of altar power drained per second while in the output phase (ritual cancels if it can't be paid).                    |
| `celestial_conditions`  | List of celestial events required (`"day"`, `"night"`, `"full_moon"`, `"new_moon"`, `"waxing"`, `"waning"`). Empty means no requirement. |
| `weather`               | Required weather conditions: `"clear"`, `"rain"`, `"storm"`. Empty means no requirement.                                     |
| `coven_count`           | Minimum number of coven members required nearby (0 means not required).                                                      |
| `require_cat`           | If `true`, requires a cat familiar (or a coven member with one) to start the ritual.                                        |

### Ritual Events

Every event shares a common `delay` field: ticks to wait, from when its turn comes up, before it starts ticking.

**Requirement**

| Type                                    | Description                                                                 | Extra fields |
|-------------------------------------------|--------------------------------------------------------------------------------|--------------|
| `witchery:consume_items`                | Waits for matching items dropped near the chalk or held in Grasspers          | `items`, `timeout` |
| `witchery:consume_items_with_particles` | Same as above, with particle feedback per item consumed                       | `items`, `timeout`, `particle_speed`, `particles_per_item` |
| `witchery:consume_sacrifice`            | Waits for and kills nearby living entities, one per slot in `entities` (see below) | `entities`, `timeout`, `drop` |

### Sacrifice Entries

`entities` is an ordered list of slots. Each slot needs one sacrifice, filled in order, so the list length is the total number of sacrifices required. A slot is one of:

| Form | Example | Meaning |
|------|---------|---------|
| Entity type | `"minecraft:pig"` | Needs exactly that entity type |
| Entity tag | `"#minecraft:raiders"` | Needs any entity in that entity type tag |
| Any-of list | `["minecraft:villager", "minecraft:cow"]` | Needs any ONE of the listed types or tags |

Any-of lists can mix types and tags, e.g. `["#c:animals", "minecraft:villager"]`. Tags use the `#` prefix and live in `data/<namespace>/tags/entity_type/`.

An any-of list is one slot, not one per option. To require several sacrifices, add several slots (e.g. three cows is three `"minecraft:cow"` entries).
**Output**

| Type                              | Description                                                                       | Extra fields                                                                                          |
|-------------------------------------|-----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| `witchery:run_command`            | Runs a command with placeholder substitution (see below)                          | `command`, `altar_power_cost`, `min_witch_power`, `max_witch_power`, `duration`, `repeat_every_ticks` |
| `witchery:summon_item`            | Drops (or floats, if `floating`) item stacks above the chalk                      | `items`, `floating`                                                                                   |
| `witchery:mirror_pair`            | Summons a bound mirror pair and demonic contract, used for the Mirror Demon ritual | -                                                                                                     |
| `witchery:summon_entities`        | Spawns entities above the chalk                                                   | `entities`, `count`, `duration`                                                                       |
| `witchery:parallel`               | Runs its child events simultaneously instead of sequentially                      | `events` (list of ritual events)                                                                      |
| `witchery:play_sound`             | Plays a sound at the chalk                                                        | `sound_id`, `volume`, `pitch`, `duration`                                                             |
| `witchery:spawn_particles`        | Spawns particles around the chalk                                                 | `particle`, `count`, `spread`, `speed`, `duration`                                                    |
| `witchery:particle_disc`          | Scatters particles outward in a flat disc                                         | `count`, `radius`, `time`, `particle`                                                                 |
| `witchery:particle_sphere`        | Scatters particles outward through a full sphere                                  | `count`, `radius`, `time`, `particle`                                                                 |
| `witchery:particle_sphere_inward` | Particles converge inward from a radius into a sphere                             | `count`, `radius`, `time`, `speed`, `particle`                                                        |
| `witchery:block_break_particle_disc` | Scatters block-break particles (matching the block below each point) in a disc    | `count`, `radius`, `time`                                                                             |
| `witchery:remove_curse`           | Removes the target player's oldest active curse                                   | -                                                                                                     |
| `witchery:bind_familiar`          | Binds a nearby unbound cat/frog/owl to a nearby player as a familiar              | `search_radius`, `search_height`                                                                      |
| `witchery:resurrect_familiar`     | Resurrects a nearby player's dead familiar                                        | `search_radius`, `search_height`                                                                      |
| `witchery:bind_spectral_creatures`| Chains nearby spectral entities to a nearby Effigy block                          | `search_radius`                                                                                       |
| `witchery:bestial_call`           | Spawns a burst of random animals around the chalk                                 | `count`, `radius`                                                                                     |
| `witchery:mine_blocks_below`      | Mines a target ore out of the column below the chalk, dropping collected items    | `target_ore_id`, `target_deepslate_ore_id`, `radius`                                                  |
| `witchery:set_midnight`           | Sets the level's time to the next midnight                                        | -                                                                                                     |

**Neutral** (ongoing/ambient effects, still have to be ordered after all requirement events, but don't themselves trigger the altar power consume or output phase transition)

| Type                        | Description                                                                                  | Extra fields |
|-------------------------------|--------------------------------------------------------------------------------------------------|--------------|
| `witchery:rot`               | Slowly zombifies nearby villagers/pigs/horses/skeletons, rots certain meats, and withers nearby crops/flowers | `effect_radius`, `transform_delay`, `duration` |
| `witchery:pull_mobs`         | Pulls nearby hostile mobs toward the chalk                                                       | `radius`, `strength`, `duration` |
| `witchery:push_mobs`         | Pushes nearby hostile mobs away from the chalk                                                   | `radius`, `strength`, `duration` |
| `witchery:raining_toad`      | Forces rain and periodically drops poisonous frogs from above for the duration                  | `duration`, `spawn_interval`, `spawn_radius`, `drop_height` |

### Command Replacements

Command strings in `witchery:run_command` support contextual placeholders, replaced at runtime:

| Placeholder               | Description                                          |
|----------------------------|------------------------------------------------------|
| `{taglockPlayer}`         | Target player whose taglock was used in the ritual   |
| `{taglockEntity}`         | Target entity whose taglock was used in the ritual   |
| `{taglockPlayerOrEntity}` | Target player OR entity, whichever taglock was used  |
| `{waystonePos}`           | Coordinates of the waystone's bound position         |
| `{ownerPos}`               | Current position of the ritual's owner               |
| `{moonPhase}`              | Current moon phase name                              |
| `{weather}`                | Current weather: `clear`, `rain`, or `storm`         |
| `{isDay}`                  | `true`/`false`, whether it's currently daytime       |
| `{dimension}`              | Dimension the ritual is running in                   |
| `{witchPower}`             | The caster's Witch Power, calculated from cat/coven  |
| `{owner}`                  | Player who started the ritual                        |
| `{chalkPos}`               | Block position of the ritual center (golden chalk)   |
| `{targetBiome}`            | Biome bound to a Biome Note used in the ritual       |

</details>

<details>
<summary>Nature Altar power JSON, directory: nature</summary>

Limit is how much power one altar can take. Power is power per block. Matches either a specific `block` or a `tag`.

```json
{
  "block": "minecraft:carrots",
  "limit": 20,
  "power": 4
}
```
or
```json
{
  "tag": "minecraft:crops",
  "limit": 20,
  "power": 4
}
```

</details>

<details>
<summary>Altar Augments JSON, directory: altar_augments</summary>

A fuller example:
```json
{
  "block": "minecraft:skeleton_skull",
  "category": "head",
  "bonus": {
    "light_bonus": 0.0,
    "head_bonus": 0.15,
    "chalice_bonus": 0.0,
    "range_multiplier": 1.0,
    "has_pentacle": false,
    "has_infinity_egg": false
  }
}
```

You don't need to fill in every field:
```json
{
  "block": "witchery:chalice",
  "bonus": {
    "chalice_bonus": 1.0
  },
  "category": "chalice"
}
```

Matches by `block` or by `tag`, and can also require specific blockstate values via `state_conditions`:
```json
{
  "tag": "witchery:candelabras",
  "bonus": {
    "light_bonus": 2.0
  },
  "category": "light",
  "state_conditions": [
    { "property": "lit", "value": "true" }
  ]
}
```

### Augment Categories

Augments are split into 5 categories. Only the best augment in each category applies, so multiple augments of the same category don't stack.

| Category | Effect                                                          |
|----------|-----------------------------------------------------------------|
| `light` | Increases **power multiplier**. Affects power generation rate   |
| `head` | Increases both **power multiplier** and **power boost**         |
| `chalice` | Increases **power boost**. Adds flat power to max capacity      |
| `range` | Multiplies the altar's detection range for nature power sources |
| `special` | Unique effects like `has_pentacle` or `has_infinity_egg`        |

### Bonus Fields

| Field | Type | Description |
|-------|------|-------------|
| `light_bonus` | double | Percentage increase to power multiplier (0.1 = 10% faster charging) |
| `head_bonus` | double | Percentage increase to both power multiplier and power boost (0.15 = 15% faster and 15% more max power) |
| `chalice_bonus` | double | Percentage increase to power boost only (0.2 = 20% more max power) |
| `range_multiplier` | double | Multiplies altar's detection range (2.0 = double range) |
| `has_pentacle` | boolean | If true, doubles the final power multiplier |
| `has_infinity_egg` | boolean | If true, multiplies power multiplier by 10 and power boost by 2 |

### State Conditions

`state_conditions` is a list of `{ "property": "<blockstate property name>", "value": "<expected value>" }` pairs. Every property listed has to match the block's current state for the augment to apply, for example requiring a candle block to be `lit=true`.

</details>

<details>
<summary>Fetish Effect JSON, directory: fetish</summary>

Maps a combination of trapped spirit counts to a fetish effect.

```json
{
  "poltergeist_count": 1,
  "banshee_count": 2,
  "specter_count": 0,
  "effect": "witchery:shrieking"
}
```

| Field | Description |
|-------|-------------|
| `poltergeist_count` | Required number of poltergeists (optional, default 0) |
| `banshee_count` | Required number of banshees (optional, default 0) |
| `specter_count` | Required number of specters (optional, default 0) |
| `effect` | The fetish effect to grant when the counts match |

</details>

<details>
<summary>Hobgoblin Trades JSON, directory: hobgoblin_trades</summary>

```json
{
  "costA": "minecraft:emerald",
  "costACount": 1,
  "costB": "witchery:spirit_of_otherwhere",
  "costBCount": 1,
  "result": "witchery:drop_of_luck",
  "resultCount": 1,
  "maxUses": 12,
  "xp": 1,
  "priceMultiplier": 0.05
}
```

</details>

<details>
<summary>Overworld Infusion metal extraction JSON, directory: overworld_infusion</summary>

Used by the Overworld infusion type's metal extraction ability: right-clicking a mapped metal block (costs 100 infusion charge) drops 2 of `item` and turns the block into `toBlock`.

```json
{
  "fromBlock": "minecraft:iron_ore",
  "toBlock": "minecraft:stone",
  "item": "minecraft:iron_ingot"
}
```

</details>

## Addon-specific data-driven stuff

<details>
<summary>Imp Trades JSON, directory: imp_trades (JAWR Forbidden Magic only)</summary>

> Requires the JAWR: Forbidden Magic addon.

Defines soul-trading offers sold by imps. `count` is optional and defaults to 1.

```json
{
  "item": "witchery:spirit_of_otherwhere",
  "count": 1,
  "soulCost": 50
}
```

| Field | Description |
|-------|-------------|
| `item` | Item granted by the trade |
| `count` | Quantity granted (optional, default 1) |
| `soulCost` | Number of souls the trade costs |

</details>

## Credits
- Model and texture of Witches oven is made by WK/AtheneNoctua.
- Texture of Taglock is made by WK/AtheneNoctua.
- Model and texture of Hunter Armor is made by TheRebelT
- Tarot Arcana Major by [starsinabox](https://starsinabox.itch.io/majorarcana)
