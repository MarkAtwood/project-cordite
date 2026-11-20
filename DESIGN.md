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
│  - SceneObject (pickups, bomb), discrete events       │
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
- **JMAP Scene** provides the spatial state layer and discrete,
  low-frequency events; gameplay traffic stays on the simulation layer

## Units and Coordinates

Cordite uses the JMAP Scene conventions so that map metadata can be
copied into Scene records without conversion: positions and distances in
meters, a right-handed Y-up coordinate system, and orientations as unit
quaternions `[x, y, z, w]` (identity `[0, 0, 0, 1]`). A yaw (turn left or
right) is a rotation about +Y: `[0, sin(θ/2), 0, cos(θ/2)]`. Times are in
seconds, speeds in meters per second, and weapon spread in degrees.

Engines with other conventions (Source and Quake units are about an inch,
Z-up; Unreal is centimeters, Z-up, left-handed) convert when loading.

## Weapon Definition Model

A weapon is a collection of numeric stats that the engine interprets:

```json
{
  "id": "standard:fps-rifle-ak47",
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
      "arms": 1.0,
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

A game mode is a state machine. An abridged form of the standard
Search and Destroy mode (the full definition is in MODES.md):

```json
{
  "id": "standard:mode-search-destroy",
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
      "format": "mr3",
      "max_rounds": 6,
      "money_reset": 10000
    }
  },

  "objectives": {
    "bomb": {
      "carrier": "random_attacker",
      "plant_time_sec": 3.2,
      "defuse_time_sec": { "no_kit": 10, "with_kit": 5 },
      "explosion_timer_sec": 40,
      "explosion_damage": 500,
      "explosion_radius": 15,
      "sites": 2
    }
  },

  "win_conditions": [
    { "type": "elimination" },
    { "type": "bomb_detonated", "winner": "attackers" },
    { "type": "bomb_defused", "winner": "defenders" },
    { "type": "time_expired", "winner": "defenders" }
  ],

  "economy": "standard:economy-competitive"
}
```

The mode does not name map zones. The bomb can be planted in the map's
zones of type `bomb_site` (see Map Metadata Model).

## Economy Model

```json
{
  "id": "standard:economy-competitive",

  "start_money": 800,
  "max_money": 16000,

  "round_win_reward": 3250,
  "round_loss_streak": [1400, 1900, 2400, 2900, 3400],
  "loss_streak_reset": "on_win",

  "kill_rewards_by_category": {
    "rifle": 300,
    "smg": 600,
    "shotgun": 900,
    "sniper": 100,
    "pistol": 300,
    "knife": 1500
  },

  "objective_rewards": {
    "bomb_plant": 300,
    "bomb_defuse": 300
  },

  "equipment": {
    "kevlar": { "price": 650, "armor": 100 },
    "kevlar_helmet": { "price": 1000, "armor": 100, "head_protection": true },
    "defuse_kit": { "price": 400, "role": "defenders" }
  }
}
```

## Map Metadata Model

Cordite does NOT define map geometry. It defines named positions and zones
in the map's region-local Scene coordinates (see Units and Coordinates);
the engine maps them to its own coordinate system:

```json
{
  "map_id": "de_dust2",
  "bounds": { "min": [-63.5, 0, -81.3], "max": [53.3, 10.2, 30.5] },

  "spawns": {
    "attackers": [
      { "position": [-35.6, 0, 12.7], "orientation": [0, 0.7071, 0, 0.7071] },
      { "position": [-34.3, 0, 11.4], "orientation": [0, 0.7071, 0, 0.7071] }
    ],
    "defenders": [
      { "position": [30.5, 0, -71.1], "orientation": [0, -0.7071, 0, 0.7071] },
      { "position": [31.8, 0, -69.9], "orientation": [0, -0.7071, 0, 0.7071] }
    ]
  },

  "zones": {
    "site_a": {
      "type": "bomb_site",
      "bounds": { "min": [17.8, 0, -73.7], "max": [27.9, 5.1, -61.0] }
    },
    "site_b": {
      "type": "bomb_site",
      "bounds": { "min": [-38.1, 0, -71.1], "max": [-22.9, 5.1, -61.0] }
    },
    "attacker_buy": {
      "type": "buy_zone",
      "team": "attackers",
      "bounds": { "min": [-40.6, 0, 7.6], "max": [-27.9, 5.1, 17.8] }
    },
    "defender_buy": {
      "type": "buy_zone",
      "team": "defenders",
      "bounds": { "min": [22.9, 0, -78.7], "max": [35.6, 5.1, -66.0] }
    }
  },

  "pickups": []
}
```

`bounds` has the shape of a Scene `SceneBounds` and becomes the
SceneRegion's `bounds` when a match region is created; each `min`
component must not exceed the matching `max` component.

A map definition does not name a SceneRegion. Region ids are assigned by
the server when a region is created and are unique only within one
account, and one map serves many matches. The binding runs the other way:
the region's `worldState` names the map (see Discovery below).

Per-team spawns are Cordite data that the game server applies. A
SceneRegion has a single `spawnPosition`/`spawnOrientation`, which is
where Scene places a new SceneAvatar record; the game server moves the
player to a team spawn in the simulation layer.

## Player Attributes Model

Speeds are in meters per second; `jump_height` and
`fall_damage_threshold` (the fall height at which damage starts) are in
meters.

```json
{
  "base_health": 100,
  "base_armor": 0,
  "max_armor": 100,

  "movement": {
    "run_speed": 6.35,
    "walk_speed": 3.3,
    "crouch_speed": 2.16,
    "jump_height": 1.41,
    "fall_damage": true,
    "fall_damage_threshold": 7.6
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

WEAPONS.md defines most of these; its "Archetypes Not Yet Defined"
section lists the rest.

Games reference these and override stats. A "realistic tactical" mode
reduces all damage-to-kill. An "arena" mode increases movement speed
and pickup respawn rates. The archetypes are starting points, not
constraints.

## JMAP Scene Integration

Section and line references below are to `draft-atwood-jmap-scene-00`
("Scene"), `draft-atwood-jmap-scene-wss-00` ("Scene WSS"), and the JMAP
Scene implementer guides, as of jmap-chat-spec commit `3445973`.

### Discovery

A Cordite match is a SceneRegion whose `worldState` carries
`corditeMode`. `SceneRegion/query` cannot filter on `worldState` (its
filters are `name`, `accessPolicy`, `viewHint`, `hasSimulationUri`,
`hasActiveAvatars`, `createdAfter`, `createdBefore`), so a client queries
with `hasSimulationUri: true`, fetches the regions with `SceneRegion/get`,
and checks `worldState.corditeMode` itself. A client sees only regions it
has access to under their `accessPolicy`; a lobby browser therefore lists
`"public"` matches and the `"invite"` matches its user is on.

A region as `SceneRegion/get` returns it (server-set `accountId`,
`ownerId`, `activeAvatarCount`, `createdAt`, and `updatedAt` omitted):

```json
{
  "id": "01JDKQ7Z3V0000000000004471",
  "name": "Competitive Match #4471",
  "description": null,
  "bounds": { "min": [-63.5, 0, -81.3], "max": [53.3, 10.2, 30.5] },
  "environment": { "gravity": 9.81 },
  "simulationUri": "udp://game.example.com:27015/match/4471",
  "inputUri": null,
  "viewHint": "3d",
  "spawnPosition": [0, 0, 0],
  "spawnOrientation": [0, 0, 0, 1],
  "accessPolicy": "invite",
  "invitedUserIds": [],
  "worldState": {
    "corditeMode": "standard:mode-search-destroy",
    "corditeVersion": "1.0",
    "corditeWeapons": "standard:fps-weapons-tactical",
    "corditeEconomy": "standard:economy-competitive",
    "status": "warmup",
    "map": "de_dust2",
    "round": 1,
    "score": { "attackers": 0, "defenders": 0 }
  }
}
```

- `id` is a ULID the server assigns on `SceneRegion/set` create; `name`
  and `bounds` are the only required create properties (Scene L1132-L1144).
  `bounds` comes from the map metadata.
- `invitedUserIds` holds the match roster. The server returns it only to
  callers who may update the region and returns `[]` to everyone else
  (Scene L529), which is what a player sees above.
- `simulationUri` is opaque to the JMAP server and untrusted; clients
  SHOULD NOT connect to it without explicit user action (Scene L415,
  L2717). The simulation guide recommends a short-lived token minted by
  the JMAP side and sent in the first message, followed by a server
  capabilities message (simulation guide L249-L256, L275); that message
  is a natural place to confirm `corditeVersion`.
- `worldState` is visible to every user who can access the region
  (Scene L538). Keep hidden information (an opponent's money or loadout)
  out of it.

### Who May Write What

Scene's permission rules decide where the game server can keep match
state:

| Record | Who may update it | Consequence for Cordite |
|--------|-------------------|-------------------------|
| SceneRegion, including `worldState` | Region owner or an administrator (Scene L1240) | The game server creates the match region, or has administrative rights, to keep score there |
| SceneObject | Its owner, the region owner, or an administrator (Scene L2699) | The game server can own pickups, the bomb, and flags |
| SceneAvatar | Only the user it belongs to; an owner or administrator may only set `leftAt` to eject (Scene L2709-L2711) | The game server cannot write a player's team, health, or money into that player's avatar `worldState` through JMAP |

Server-side reconciliation (avatar positions copied from the simulation
layer) is done by the deployment against its own store, not as a JMAP
caller (Scene L2653-L2660; simulation guide Architecture B, L1262).

Players map onto the SceneAvatar lifecycle:

- **Join:** `SceneAvatar/set` create with `regionId`. A user has at most
  one record per region; a second create fails with `alreadyExists` and
  `existingId` (Scene L689-L690, L2317).
- **Leave:** an update that sets `leftAt`. The server stores its own time
  and echoes it in `updated` (Scene L2385-L2447).
- **Rejoin after a disconnect:** an update that sets `leftAt` to `null` on
  the same record (Scene L2448). The server may set `leftAt` itself when
  the connection drops.
- **Kick:** the region owner or an administrator sets `leftAt` on the
  player's record (eject); the update may patch nothing else.

### Real-Time Events

Gameplay traffic (input, movement, shots, hits, damage) belongs to the
simulation layer behind `simulationUri`. The JMAP WebSocket carries only
discrete low-frequency events, rate-limited to two `SceneObjectEvent`
updates per object per second and five `SceneInteractionEvent`s per user
per region per second (Scene WSS L694-L712).

Ephemeral events require the separate capability
`urn:ietf:params:jmap:scene:websocket` and a `SceneStreamEnable` message
on the JMAP WebSocket (Scene WSS L135-L152, L432). Scene WSS defines
three event types and none of them is a server-originated announcement.
How Cordite carries the kill feed and round transitions is an open
question (see Open Questions). The candidates are:

- **`SceneInteractionEvent`**, as a user acting on an interactable
  object. The server sets `userId` to the submitting user and ignores any
  supplied value, and accepts the event only from a user with an active
  avatar in the region, about a SceneObject in that region with
  `interactable: true` (Scene WSS L297-L300, L413-L419; Scene L2137). If
  the game server relays kills this way, it needs its own avatar in the
  region and an interactable "match feed" object, `userId` names the game
  server rather than the killer, and a busy moment can exceed five events
  a second.
- **`SceneObjectEvent`**, raised when the game server updates a match-state
  SceneObject's `worldState`. Subscribers then call `SceneObject/get`;
  updates coalesce at two per second.
- **`StateChange`** after a `SceneRegion.worldState` update (score, round,
  status), followed by `SceneRegion/get`.
- **Chat system messages** in a Chat bound through `chatId`, as the
  JMAP Scene games guide suggests for move logs.

A kill relayed as an interaction would look like this (the `action` uses
reverse-domain notation as Scene WSS L306 recommends; the domain is an
open question):

```json
{
  "@type": "SceneInteractionEvent",
  "accountId": "account-xyz",
  "regionId": "01JDKQ7Z3V0000000000004471",
  "objectId": "01JDKQ7Z3V0000000000000F33",
  "userId": "user:gameserver@game.example.com",
  "action": "io.github.markatwood.cordite.kill",
  "data": {
    "killer": "user:alice@example.com",
    "victim": "user:bob@example.com",
    "weapon": "standard:fps-rifle-ak47",
    "headshot": true,
    "assist": "user:carol@example.com"
  }
}
```

`accountId` is required on every Scene WSS event because region and
object ids are unique only within an account (Scene WSS L181).

### Spectators and Companion Apps

A non-engine client (lobby browser, stream overlay, mobile companion) can
use Scene without running the game engine, within these limits:

- **State** (`SceneRegion.worldState`, SceneObjects, SceneAvatars) needs
  only access to the region: `/get`, `/changes`, and `StateChange` push.
- **Ephemeral events** go only to users with an active avatar in the
  region (Scene WSS L689, L1186). A spectator who wants the event stream
  has to enter the region as a SceneAvatar, which counts toward
  `activeAvatarCount` and `maxAvatarsPerRegion` and announces the entry
  with a `SceneAvatarEvent`. Scene has no spectator role; the game server
  tells players from spectators by its own roster.
- **Positions** in SceneAvatar records come from reconciliation, which
  Scene RECOMMENDS every 5 to 30 seconds (Scene L2657). That is a fallback
  for late joiners, not a live minimap; a live spectator view needs the
  simulation layer.
- **Hidden information.** Any user with access to the region can read
  every SceneAvatar's last position. In a competitive round that is a
  wallhack. Scene's visibility filtering is defined for SceneObjects
  (Scene L2662-L2676, L2753), not for avatars, so a Cordite deployment
  should not reconcile living players' positions into Scene during a live
  round, or should delay or coarsen them (Scene L2737).

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
