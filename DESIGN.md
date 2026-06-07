# Cordite — Design Document

## Problem Statement

Every FPS reinvents the same rules layer: weapon damage tables, game mode
logic, economy systems, round structures. These are defined in engine-
specific code (Unreal Blueprints, Unity C#, custom scripts) that can't
be shared, analyzed, or validated independently of the engine.

Cordite extracts the declarative parts — the parts that ARE data — into
a standard JSON schema, leaving the hard real-time problems to the engine.

## Scope: What's Declarative and What Isn't

The dividing line is simple: if it's a number in a config file or a rule
that could be expressed as a state machine, it's Cordite's job. If it
requires continuous simulation, it's the engine's job.

### Cordite's scope (data, not code)

| Layer | Examples | Why it's declarative |
|-------|----------|---------------------|
| Weapon stats | Damage, fire rate, spread, falloff | Numbers in a table |
| Game modes | Deathmatch, CTF, bomb defusal | State machines with clear transitions |
| Economy | Buy menus, kill rewards, loss bonuses | Arithmetic rules |
| Round structure | Win conditions, side swap, overtime | Finite state |
| Pickups | Health packs, ammo, power-ups | Type + amount + respawn timer |
| Map metadata | Spawn points, buy zones, objectives | Named positions |
| Player classes | Health, armor, speed per class | Numbers in a table |
| Damage model | Hitbox multipliers, armor reduction | Arithmetic rules |
| Vehicle rules | Damage, fuel, pit stops, race structure, flight envelope limits | Numbers and state machines |

### Engine's scope (not Cordite)

| Layer | Why it can't be declarative |
|-------|-----------------------------|
| Physics | Continuous simulation at 20-128 Hz |
| Hit detection | Requires lag compensation, tick interpolation, ray/projectile tracing |
| Netcode | Client prediction, server reconciliation, bandwidth management |
| Map geometry | 3D mesh data, BSP trees, collision volumes, navmeshes |
| Rendering | Shaders, particles, lighting, LOD, culling |
| Audio | Spatial audio, occlusion, HRTF |
| Animation | Skeletal meshes, blend trees, IK |
| AI | Bot navigation, decision making |
| Input | HID mapping, axis curves, dead zones |

## Architecture

```
┌──────────────────────────────────────────────────────┐
│  Cordite game mode definition (JSON)                  │
│  - weapon stats, round rules, economy, objectives     │
├──────────────────────────────────────────────────────┤
│  JMAP Scene (spatial state layer)                     │
│  - SceneRegion (map), SceneAvatar (players),          │
│  - SceneObject (weapons/pickups), events (kill feed)  │
├──────────────────────────────────────────────────────┤
│  Game server (simulationUri, engine-specific)         │
│  - Physics, hit detection, netcode                    │
│  - Reads Cordite schema for rules                     │
│  - Authoritative for all game state                   │
├──────────────────────────────────────────────────────┤
│  Game client (engine-specific)                        │
│  - Rendering, input, prediction, audio                │
│  - Reads Cordite schema for HUD, buy menu, weapon UI  │
└──────────────────────────────────────────────────────┘
```

The Cordite JSON travels to both client and server:
- **Server** reads it to configure game rules (damage calc, round logic, economy)
- **Client** reads it to render HUD, buy menus, weapon stats, game mode UI
- **JMAP Scene** provides the spatial state layer and real-time events

## Weapon Definition Model

A weapon is a collection of numeric stats that the engine interprets:

```json
{
  "id": "rifle_ak47",
  "name": "AK-47",
  "category": "rifle",
  "slot": "primary",
  "price": 2700,
  "kill_reward": 300,

  "damage": {
    "base": 36,
    "armor_penetration": 0.775,
    "range_falloff": {
      "start_distance": 15,
      "end_distance": 35,
      "min_multiplier": 0.75
    },
    "hitbox_multipliers": {
      "head": 4.0,
      "chest": 1.0,
      "stomach": 1.25,
      "legs": 0.75
    }
  },

  "fire": {
    "mode": "auto",
    "rate_rpm": 600,
    "burst_count": null
  },

  "accuracy": {
    "standing_spread": 0.6,
    "moving_spread": 3.5,
    "crouching_spread": 0.4,
    "first_shot_accurate": true,
    "recoil_pattern": "standard:ak47_recoil"
  },

  "magazine": {
    "size": 30,
    "reserve": 90,
    "reload_time_sec": 2.5,
    "reload_type": "magazine"
  },

  "mobility": {
    "speed_multiplier": 0.915,
    "draw_time_sec": 1.0
  }
}
```

## Game Mode Model

A game mode is a state machine:

```json
{
  "id": "search_and_destroy",
  "name": "Search and Destroy",
  "team_based": true,
  "teams": {
    "count": 2,
    "roles": ["attackers", "defenders"],
    "swap_roles_at_round": 12
  },

  "players": {
    "per_team": 5,
    "respawn": false,
    "friendly_fire": true
  },

  "rounds": {
    "wins_needed": 13,
    "time_limit_sec": 115,
    "freeze_time_sec": 15,
    "overtime": {
      "trigger": "tied_at_12",
      "max_rounds": 6,
      "money_reset": 10000
    }
  },

  "objectives": {
    "bomb": {
      "plant_time_sec": 3.2,
      "defuse_time_sec": { "no_kit": 10, "with_kit": 5 },
      "explosion_timer_sec": 40,
      "explosion_damage": 500,
      "explosion_radius": 15,
      "plant_zones": ["site_a", "site_b"]
    }
  },

  "win_conditions": [
    { "type": "elimination", "description": "All opponents eliminated" },
    { "type": "objective_complete", "role": "attackers", "description": "Bomb detonated" },
    { "type": "objective_defended", "role": "defenders", "description": "Time expired or bomb defused" }
  ]
}
```

## Economy Model

```json
{
  "start_money": 800,
  "max_money": 16000,

  "round_win_reward": 3250,
  "round_loss_streak": [1400, 1900, 2400, 2900, 3400],
  "loss_streak_reset": "on_win",

  "kill_rewards": {
    "rifle": 300,
    "smg": 600,
    "shotgun": 900,
    "sniper": 100,
    "pistol": 300,
    "knife": 1500
  },

  "bomb_plant_reward": 300,

  "equipment": {
    "kevlar": { "price": 650, "armor": 100 },
    "kevlar_helmet": { "price": 1000, "armor": 100, "head_protection": true },
    "defuse_kit": { "price": 400, "role": "defenders" }
  }
}
```

## Map Metadata Model

Cordite does NOT define map geometry. It defines named positions and zones
that the engine maps to its own coordinate system:

```json
{
  "map_id": "de_dust2",
  "region_id": "01J5ABC0000000000000000001",
  "bounds": { "min": [-2500, -1200, 0], "max": [2100, 3200, 400] },

  "spawns": {
    "attackers": [
      { "position": [-1400, -500, 0], "orientation": [0, 0, 0.7, 0.7] },
      { "position": [-1350, -450, 0], "orientation": [0, 0, 0.7, 0.7] }
    ],
    "defenders": [
      { "position": [1200, 2800, 0], "orientation": [0, 0, -0.7, 0.7] },
      { "position": [1250, 2750, 0], "orientation": [0, 0, -0.7, 0.7] }
    ]
  },

  "zones": {
    "site_a": {
      "type": "bomb_site",
      "bounds": { "min": [700, 2400, 0], "max": [1100, 2900, 200] }
    },
    "site_b": {
      "type": "bomb_site",
      "bounds": { "min": [-1500, 2400, 0], "max": [-900, 2800, 200] }
    },
    "attacker_buy": {
      "type": "buy_zone",
      "team": "attackers",
      "bounds": { "min": [-1600, -700, 0], "max": [-1100, -300, 200] }
    },
    "defender_buy": {
      "type": "buy_zone",
      "team": "defenders",
      "bounds": { "min": [900, 2600, 0], "max": [1400, 3100, 200] }
    }
  },

  "pickups": []
}
```

## Player Attributes Model

```json
{
  "base_health": 100,
  "base_armor": 0,
  "max_armor": 100,

  "movement": {
    "run_speed": 250,
    "walk_speed": 130,
    "crouch_speed": 85,
    "jump_height": 55.5,
    "fall_damage": true,
    "fall_damage_threshold": 300
  },

  "hitbox": {
    "model": "standard:fps_hitbox_5zone",
    "zones": ["head", "chest", "stomach", "arms", "legs"]
  },

  "inventory": {
    "slots": {
      "primary": 1,
      "secondary": 1,
      "melee": 1,
      "grenades": { "max": 4, "max_per_type": { "smoke": 1, "flash": 2, "frag": 1, "incendiary": 1 } },
      "equipment": 1
    }
  }
}
```

## Standard Weapon Archetypes

Like Baize's component registry, Cordite defines standard weapon archetypes
that game modes can reference or extend:

```
standard:fps-pistol-9mm        — baseline sidearm (Glock/P2000 class)
standard:fps-pistol-deagle     — high-damage hand cannon
standard:fps-smg-mp5           — balanced SMG
standard:fps-smg-p90           — high-capacity SMG
standard:fps-rifle-ak47        — high damage, hard recoil
standard:fps-rifle-m4          — balanced assault rifle
standard:fps-sniper-awp        — one-shot bolt-action
standard:fps-sniper-scout      — mobile semi-auto sniper
standard:fps-shotgun-pump      — high close-range damage
standard:fps-shotgun-auto      — faster fire, less damage
standard:fps-lmg-negev         — high capacity, slow movement
standard:fps-knife             — melee, always available
standard:fps-grenade-frag      — area damage
standard:fps-grenade-flash     — blind/deafen
standard:fps-grenade-smoke     — area denial (visual)
standard:fps-grenade-incendiary — area denial (damage)
standard:fps-grenade-decoy     — false radar signature
```

Games reference these and override stats. A "realistic tactical" mode
reduces all damage-to-kill. An "arena" mode increases movement speed
and pickup respawn rates. The archetypes are starting points, not
constraints.

## JMAP Scene Integration

### Discovery

A client discovers a Cordite game by finding a SceneRegion with
`customProperties.corditeMode`:

```json
{
  "id": "region-match-001",
  "name": "Competitive Match #4471",
  "simulationUri": "wss://game.example.com/match/4471",
  "customProperties": {
    "corditeMode": "search_and_destroy",
    "corditeVersion": "1.0",
    "corditeWeapons": "standard:fps-weapons-tactical",
    "corditeEconomy": "standard:fps-economy-competitive",
    "status": "warmup",
    "map": "de_dust2",
    "round": 1,
    "score": { "attackers": 0, "defenders": 0 }
  }
}
```

### Real-Time Events

Game events flow through SceneInteractionEvent (via JMAP Scene WSS):

```json
{
  "@type": "SceneInteractionEvent",
  "regionId": "region-match-001",
  "objectId": null,
  "userId": "user:alice@example.com",
  "action": "cordite.kill",
  "data": {
    "victim": "user:bob@example.com",
    "weapon": "rifle_ak47",
    "headshot": true,
    "assist": "user:carol@example.com"
  }
}
```

These events drive spectator UI, kill feed, stats tracking, and JMAP
Scene clients that aren't running the full game engine (lobby viewers,
mobile companion apps, stream overlays).

### Spectator Mode

JMAP Scene provides the spectator layer. A non-engine client connected
to the SceneRegion sees:

- SceneAvatar positions (updated at low Hz by the game server)
- SceneInteractionEvents (kills, objectives, round transitions)
- SceneRegion.customProperties (score, round, status)

This is enough for a lobby browser, stream overlay, or mobile
companion without running the game engine.

## What Cordite Enables

1. **Game mode modding as data.** Change weapon stats, round rules, or
   economy by editing JSON. No recompilation, no engine access needed.

2. **Cross-engine game modes.** The same Cordite definition works on any
   engine that reads it. Balance patches are engine-independent.

3. **Spectator/companion apps.** JMAP Scene clients can show match state,
   kill feeds, and economy without the game engine.

4. **Stats and analytics.** Parse game mode definitions to compute
   theoretical TTK, DPS, economy curves without running the game.

5. **Anti-cheat baseline.** The server reads the same weapon stats as the
   client. Discrepancies between declared stats and observed behavior
   flag potential cheats.

6. **Physical-world consumers.** Nothing in Cordite assumes a screen.
   A drone running CTF objective logic, a robot reading weapon stats
   for a paintball turret, or a vehicle using zone and waypoint metadata
   are all valid Cordite consumers. The same JSON that configures a
   video game configures a physical arena.

## Reference Engine (separate project)

Cordite is a spec. It does not include a game engine. But a spec without
an engine means people still have to build one before they can play.

The intended companion is a **reference engine** — a separate project that
consumes Cordite JSON definitions and provides the hard real-time layer:

```
Cordite (spec)
    ↑ reads (depends on)
    │
Reference engine (separate project, separate repo)
    - Physics simulation (rigid body, projectiles, grenades)
    - Netcode (client prediction, server reconciliation, lag compensation)
    - Hit detection (server-authoritative, hitbox vs hitscan vs projectile)
    - Map loading (glTF geometry, collision meshes, navmeshes)
    - Audio (spatial, occlusion)
    - Rendering (or delegates to client engine)
```

The dependency is one-way: the engine reads Cordite definitions, Cordite
does not know about the engine. Someone could build a different engine that
also reads Cordite JSON — an Unreal plugin, a Godot module, a custom
Rust server, or an O3DE Gem — and it would be equally valid.

The goal: a game designer provides **art assets** (character models, weapon
models, map geometry as glTF, sounds, textures) plus a **Cordite JSON**
(weapon stats, game mode, economy), and the reference engine runs a
playable game. No engine programming required.

The reference engine is a separate project with its own name and repo.
Cordite does not prescribe its architecture, language, or implementation.

### Candidate Platforms

- **O3DE** (Open 3D Engine) — Amazon Lumberyard rewritten and donated to
  the Linux Foundation's Open 3D Foundation. Apache 2.0 licensed.
  Component-based architecture with a "Gem" extension system; a Cordite
  reader would be a Gem. Already data-driven (prefabs, script canvas) and
  has a built-in server-authoritative multiplayer framework. Heavy — tens
  of millions of lines — but feature-complete.
- **Godot** — MIT licensed, lighter weight, active community. GDScript or
  C++ modules could consume Cordite JSON.
- **Custom Rust server** — Minimal, purpose-built. Fastest path to a
  headless game server that reads Cordite JSON and runs authoritative
  simulation, but requires building rendering/audio separately.

## Open Questions

1. **Recoil patterns.** Are these data (a sequence of (dx, dy) offsets per
   shot) or engine-specific? Probably data — store as an array of vectors.

2. **Ability/skill systems.** Hero shooters (Overwatch, Valorant) have
   per-character abilities with cooldowns, charges, and complex effects.
   How much of this is declarative vs. needs engine code?

3. **Vehicle definitions.** Games with vehicles (Battlefield, Halo) need
   vehicle stats (speed, health, weapon mounts). Same pattern as weapons?
   Flight sims and racing sims are in scope — if a full HOTAS or
   wheel/pedal input deck can't drive a Cordite game, the abstraction
   is wrong. Vehicle physics stay in the engine; vehicle *rules* (damage
   model, fuel, pit stops, race structure, flight envelope limits) are
   declarative data.

4. **Damage types.** Some games have damage type interactions (fire vs.
   ice, armor-piercing vs. standard). Is this a simple multiplier table
   or does it need engine code?

5. **Progression/unlocks.** Are XP, ranks, and unlock trees part of the
   game mode definition, or a platform concern above Cordite?

6. **Map rotation.** Is the playlist/map rotation part of Cordite or a
   server configuration concern?
