# Cordite — Standard Weapon Definitions

## Weapon Schema

Every weapon has the same structure. Engines read these fields to configure
their weapon systems.

### Required Fields

```json
{
  "id": "string",
  "name": "string",
  "category": "pistol | smg | rifle | sniper | shotgun | lmg | melee | grenade | equipment",
  "slot": "primary | secondary | melee | grenade | equipment",
  "price": "number (0 = free)",

  "damage": {
    "base": "number (HP per hit)",
    "armor_penetration": "number (0.0-1.0, fraction that bypasses armor)",
    "hitbox_multipliers": {
      "head": "number",
      "chest": "number",
      "stomach": "number",
      "arms": "number",
      "legs": "number"
    }
  },

  "fire": {
    "mode": "auto | semi | burst | bolt | pump | melee | thrown",
    "rate_rpm": "number (rounds per minute)",
    "burst_count": "number | null"
  },

  "magazine": {
    "size": "number",
    "reserve": "number",
    "reload_time_sec": "number",
    "reload_type": "magazine | single_shell | belt"
  }
}
```

### Optional Fields

```json
{
  "damage": {
    "range_falloff": {
      "start_distance": "number (meters, full damage before this)",
      "end_distance": "number (meters, minimum damage after this)",
      "min_multiplier": "number (0.0-1.0)"
    },
    "pellets": "number (shotguns: damage is per-pellet)",
    "headshot_lethal_range": "number (meters, one-shot-kill range for snipers)"
  },

  "accuracy": {
    "standing_spread": "number (degrees)",
    "moving_spread": "number",
    "crouching_spread": "number",
    "jumping_spread": "number",
    "first_shot_accurate": "boolean",
    "recoil_pattern": "string (reference to pattern data)"
  },

  "mobility": {
    "speed_multiplier": "number (1.0 = no penalty)",
    "draw_time_sec": "number",
    "ads_time_sec": "number (aim-down-sights time)"
  },

  "scope": {
    "zoom_levels": "[number] (magnification factors)",
    "scope_type": "none | holo | acog | sniper"
  }
}
```

## Standard Weapon Definitions

### Pistols

```json
{
  "id": "standard:fps-pistol-9mm",
  "name": "9mm Pistol",
  "category": "pistol",
  "slot": "secondary",
  "price": 200,
  "damage": {
    "base": 30,
    "armor_penetration": 0.47,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 15, "end_distance": 35, "min_multiplier": 0.8 }
  },
  "fire": { "mode": "semi", "rate_rpm": 400 },
  "accuracy": { "standing_spread": 0.4, "moving_spread": 1.8, "first_shot_accurate": true },
  "magazine": { "size": 20, "reserve": 120, "reload_time_sec": 2.2, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 1.0, "draw_time_sec": 0.5 }
}
```

```json
{
  "id": "standard:fps-pistol-deagle",
  "name": "Hand Cannon",
  "category": "pistol",
  "slot": "secondary",
  "price": 700,
  "damage": {
    "base": 63,
    "armor_penetration": 0.932,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 20, "end_distance": 40, "min_multiplier": 0.6 }
  },
  "fire": { "mode": "semi", "rate_rpm": 267 },
  "accuracy": { "standing_spread": 0.6, "moving_spread": 5.0, "first_shot_accurate": true },
  "magazine": { "size": 7, "reserve": 35, "reload_time_sec": 2.2, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.96, "draw_time_sec": 0.75 }
}
```

### SMGs

```json
{
  "id": "standard:fps-smg-mp5",
  "name": "MP5",
  "category": "smg",
  "slot": "primary",
  "price": 1500,
  "damage": {
    "base": 27,
    "armor_penetration": 0.575,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 10, "end_distance": 25, "min_multiplier": 0.7 }
  },
  "fire": { "mode": "auto", "rate_rpm": 750 },
  "accuracy": { "standing_spread": 0.5, "moving_spread": 2.0, "first_shot_accurate": true },
  "magazine": { "size": 30, "reserve": 120, "reload_time_sec": 2.4, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.97, "draw_time_sec": 0.7 }
}
```

### Rifles

```json
{
  "id": "standard:fps-rifle-ak47",
  "name": "AK-47",
  "category": "rifle",
  "slot": "primary",
  "price": 2700,
  "damage": {
    "base": 36,
    "armor_penetration": 0.775,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 15, "end_distance": 35, "min_multiplier": 0.75 }
  },
  "fire": { "mode": "auto", "rate_rpm": 600 },
  "accuracy": { "standing_spread": 0.6, "moving_spread": 3.5, "crouching_spread": 0.4, "first_shot_accurate": true },
  "magazine": { "size": 30, "reserve": 90, "reload_time_sec": 2.5, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.915, "draw_time_sec": 1.0 }
}
```

```json
{
  "id": "standard:fps-rifle-m4",
  "name": "M4A1",
  "category": "rifle",
  "slot": "primary",
  "price": 3100,
  "damage": {
    "base": 33,
    "armor_penetration": 0.70,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 15, "end_distance": 35, "min_multiplier": 0.8 }
  },
  "fire": { "mode": "auto", "rate_rpm": 666 },
  "accuracy": { "standing_spread": 0.4, "moving_spread": 3.0, "crouching_spread": 0.3, "first_shot_accurate": true },
  "magazine": { "size": 30, "reserve": 90, "reload_time_sec": 3.1, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.94, "draw_time_sec": 1.0 }
}
```

### Snipers

```json
{
  "id": "standard:fps-sniper-awp",
  "name": "AWP",
  "category": "sniper",
  "slot": "primary",
  "price": 4750,
  "damage": {
    "base": 115,
    "armor_penetration": 0.975,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 }
  },
  "fire": { "mode": "bolt", "rate_rpm": 41 },
  "accuracy": { "standing_spread": 0.1, "moving_spread": 8.0, "first_shot_accurate": true },
  "magazine": { "size": 5, "reserve": 30, "reload_time_sec": 3.67, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.82, "draw_time_sec": 1.5 },
  "scope": { "zoom_levels": [4.0], "scope_type": "sniper" }
}
```

```json
{
  "id": "standard:fps-sniper-scout",
  "name": "Scout",
  "category": "sniper",
  "slot": "primary",
  "price": 1700,
  "damage": {
    "base": 75,
    "armor_penetration": 0.85,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 }
  },
  "fire": { "mode": "bolt", "rate_rpm": 48 },
  "accuracy": { "standing_spread": 0.1, "moving_spread": 2.5, "first_shot_accurate": true },
  "magazine": { "size": 10, "reserve": 90, "reload_time_sec": 3.0, "reload_type": "magazine" },
  "mobility": { "speed_multiplier": 0.96, "draw_time_sec": 1.0 },
  "scope": { "zoom_levels": [2.5], "scope_type": "sniper" }
}
```

### Shotguns

```json
{
  "id": "standard:fps-shotgun-pump",
  "name": "Pump Shotgun",
  "category": "shotgun",
  "slot": "primary",
  "price": 1200,
  "damage": {
    "base": 26,
    "pellets": 9,
    "armor_penetration": 0.50,
    "hitbox_multipliers": { "head": 4.0, "chest": 1.0, "stomach": 1.25, "arms": 1.0, "legs": 0.75 },
    "range_falloff": { "start_distance": 5, "end_distance": 15, "min_multiplier": 0.3 }
  },
  "fire": { "mode": "pump", "rate_rpm": 68 },
  "accuracy": { "standing_spread": 3.0, "moving_spread": 5.0 },
  "magazine": { "size": 7, "reserve": 32, "reload_time_sec": 0.55, "reload_type": "single_shell" },
  "mobility": { "speed_multiplier": 0.95, "draw_time_sec": 0.8 }
}
```

### Melee

```json
{
  "id": "standard:fps-knife",
  "name": "Knife",
  "category": "melee",
  "slot": "melee",
  "price": 0,
  "damage": {
    "base": 40,
    "armor_penetration": 1.0,
    "hitbox_multipliers": { "head": 1.0, "chest": 1.0, "stomach": 1.0, "arms": 1.0, "legs": 1.0 },
    "backstab_multiplier": 2.5
  },
  "fire": { "mode": "melee", "rate_rpm": 120 },
  "mobility": { "speed_multiplier": 1.0, "draw_time_sec": 0.5 }
}
```

### Grenades

```json
{
  "id": "standard:fps-grenade-frag",
  "name": "Frag Grenade",
  "category": "grenade",
  "slot": "grenade",
  "price": 300,
  "damage": { "base": 98, "radius": 7.5, "falloff": "linear" },
  "fire": { "mode": "thrown" },
  "fuse_time_sec": 2.0,
  "max_carry": 1
}
```

```json
{
  "id": "standard:fps-grenade-flash",
  "name": "Flashbang",
  "category": "grenade",
  "slot": "grenade",
  "price": 200,
  "effect": { "type": "blind", "duration_sec": 3.5, "radius": 10, "falloff": "angle_and_distance" },
  "fire": { "mode": "thrown" },
  "fuse_time_sec": 1.5,
  "max_carry": 2
}
```

```json
{
  "id": "standard:fps-grenade-smoke",
  "name": "Smoke Grenade",
  "category": "grenade",
  "slot": "grenade",
  "price": 300,
  "effect": { "type": "smoke", "radius": 8, "duration_sec": 18, "blocks_los": true },
  "fire": { "mode": "thrown" },
  "fuse_time_sec": 3.0,
  "max_carry": 1
}
```

```json
{
  "id": "standard:fps-grenade-incendiary",
  "name": "Incendiary Grenade",
  "category": "grenade",
  "slot": "grenade",
  "price": 600,
  "effect": { "type": "fire", "radius": 5, "duration_sec": 7, "damage_per_sec": 40 },
  "fire": { "mode": "thrown" },
  "fuse_time_sec": 2.0,
  "max_carry": 1
}
```

## Extending Standard Weapons

Override specific fields:

```json
{
  "extends": "standard:fps-rifle-ak47",
  "id": "custom:ak47-nerfed",
  "damage": { "base": 30 },
  "price": 2500
}
```

Only overridden fields change. Everything else inherits.
