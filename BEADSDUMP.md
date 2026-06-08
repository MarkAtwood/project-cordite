# Beads Issue Dump

## cordite-0ij - Vehicle definitions: stats schema for flight/race sims
- **Status:** OPEN
- **Priority:** P2
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Games with vehicles (Battlefield, Halo) and dedicated sims (DCS, iRacing, MSFS) need vehicle stats. Same pattern as weapons (numbers in a table) but much broader: speed, health, weapon mounts, fuel, pit stops, race structure, flight envelope limits. Design test: a full HOTAS or wheel/pedal input deck must be able to drive a Cordite game. Vehicle physics stay in the engine; vehicle rules are declarative. Need to design the schema.

---

## cordite-ano - Recoil patterns: data or engine-specific?
- **Status:** OPEN
- **Priority:** P3
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Are recoil patterns (sequence of (dx,dy) offsets per shot) declarative data that belongs in Cordite, or engine-specific? Probably data — store as an array of vectors. But need to decide: fixed pattern vs. procedural with seed? Per-weapon or per-archetype? How does the engine interpret the pattern (exact replay vs. approximation)?

---

## cordite-hr6 - Ability/skill systems: how much is declarative?
- **Status:** OPEN
- **Priority:** P3
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Hero shooters (Overwatch, Valorant) have per-character abilities with cooldowns, charges, and complex effects. How much of this is declarative (cooldown timers, charge counts, damage numbers) vs. needs engine code (projectile trajectories, area effects, custom physics)? Where is the line between 'numbers in a table' and 'needs a WASM module or engine plugin'?

---

## cordite-kbh - Damage types: multiplier table or engine code?
- **Status:** OPEN
- **Priority:** P3
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Some games have damage type interactions (fire vs. ice, armor-piercing vs. standard, explosive vs. kinetic). Is this a simple multiplier table (DamageType x ArmorType -> multiplier) that Cordite can express, or does it need engine code for complex interactions (DOT stacking, resistance debuffs, elemental combos)? Probably a tiered answer: simple multiplier table is Cordite, complex interactions are engine.

---

## cordite-m5p - Map rotation: Cordite or server config?
- **Status:** OPEN
- **Priority:** P4
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Is the playlist/map rotation (which maps in what order, voting, veto, random selection) part of Cordite or a server configuration concern? Arguments for Cordite: it's a finite state machine, which is Cordite's strength. Arguments against: it's deployment-specific (ranked vs. casual vs. custom), tied to matchmaking, and changes without changing the game mode. Probably server config, but Cordite could define a standard schema that servers optionally consume.

---

## cordite-n0z - Progression/unlocks: Cordite or platform concern?
- **Status:** OPEN
- **Priority:** P4
- **Type:** task
- **Owner:** Mark Atwood
- **Labels:** human-judgment
- **Created:** 2026-06-07

**Description:**
Are XP, ranks, unlock trees, battle passes, and seasonal content part of the game mode definition, or a platform concern above Cordite? Arguments for Cordite: unlock trees affect available weapons/equipment, which Cordite already defines. Arguments against: progression is account-level state, not match-level state; it's a monetization/retention layer, not a rules layer. Probably out of scope — Cordite defines what's available, the platform decides who has unlocked it.
