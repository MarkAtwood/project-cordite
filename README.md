# Cordite

A declarative game mode and weapon definition language for real-time
first-person shooters, built on JMAP Scene. Defines the rules layer
(weapons, economy, rounds, objectives) as data, leaving physics,
netcode, and hit detection to the engine where they belong.

**Cordite** is the smokeless propellant in every cartridge — present in
every shot, belonging to no weapon. Like Baize (the felt under the cards),
it's the invisible medium that makes the game work.

## Relationship to Baize

[Baize](https://github.com/MarkAtwood/project-baize) is a declarative
schema for turn-based board and card games. Cordite applies similar
thinking to real-time shooters but is a much thinner spec — FPS games
have hard real-time problems (physics, netcode, hit detection) that
can't be declarative. Cordite only covers the rules layer on top.

| | Baize | Cordite |
|--|-------|---------|
| Game type | Turn-based | Real-time |
| Core engine | Declarative + WASM | Engine-specific (Unreal, Unity, Godot, custom) |
| Server role | Sequencer + hidden state | Authoritative simulation |
| Transport | WebSocket | Deployment-chosen behind `simulationUri` (typically UDP or QUIC) |
| Scene binding | Optional | Required |

## Relationship to JMAP Scene

Cordite requires `urn:ietf:params:jmap:scene`. Ephemeral events also
need `urn:ietf:params:jmap:scene:websocket`. The mapping:

| Cordite concept | JMAP Scene |
|----------------|-----------|
| Match on a map | SceneRegion (bounds, environment, simulationUri) |
| Match state (mode, round, score) | SceneRegion.worldState |
| Players | SceneAvatar (one record per user per region) |
| Dropped weapons, pickups, bomb, flags | SceneObject (with worldState) |
| Kill feed, round events | Open question; see DESIGN.md "JMAP Scene Integration" |
| The game server | The endpoint at SceneRegion.simulationUri |

Weapon, mode, and economy *definitions* are Cordite JSON, not Scene
records. A weapon in a player's hands is not a SceneObject either; it is
player state.

## What Cordite defines (declarative)

- Weapon stats (damage, fire rate, spread, reload, magazine size)
- Game mode rules (deathmatch, CTF, search-and-destroy, etc.)
- Economy systems (buy rounds, kill rewards, loss streaks)
- Round/match structure (round count, side swap, overtime)
- Pickup/item definitions (health, armor, ammo, power-ups)
- Map metadata (spawn points, objectives, buy zones — not geometry)
- Player attributes (health, armor, speed, hitbox classes)

## What Cordite does NOT define (engine-specific)

- Map geometry and collision (use glTF, BSP, engine-native formats)
- Physics simulation (engine-specific, runs at 20-128 tick/sec)
- Netcode (client prediction, lag compensation, interpolation)
- Hit detection (server-authoritative, tick-dependent)
- Rendering (shaders, particles, post-processing)
- Audio (spatial audio, HRTF)

## Status

Design phase. No implementation yet.

## Key Documents

- `DESIGN.md` — Architecture, scope, what's declarative and what isn't
- `WEAPONS.md` — Standard weapon archetypes and stat definitions
- `MODES.md` — Game mode definitions (deathmatch, CTF, S&D, etc.) and
  the standard economy
- `EXAMPLES.md` — Example game mode definitions (planned, not yet written)

## Schema Format

JSON (same decision as Baize, same rationale: zero-dependency parsing).

## License

Same three-tier structure as Baize:

- **Specification** — [CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
- **Client-embeddable engine** — [MIT](https://opensource.org/licenses/MIT)
- **Servers and standalone clients** — [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html)

**Your game mode definitions** are yours to license however you want.
But if clients and servers can't read them, nobody can play.
