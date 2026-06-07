# Cordite — Standard Game Modes

Standard game mode definitions that any Cordite-compatible engine can run.
Games reference these by identifier and override specific values.

## Deathmatch

```json
{
  "id": "standard:mode-deathmatch",
  "name": "Free-for-All Deathmatch",
  "team_based": false,

  "players": {
    "min": 2,
    "max": 16,
    "respawn": true,
    "respawn_delay_sec": 3,
    "spawn_protection_sec": 2
  },

  "end_conditions": [
    { "type": "score_limit", "frags": 50 },
    { "type": "time_limit", "minutes": 10 }
  ],

  "scoring": {
    "kill": 1,
    "suicide": -1,
    "team_kill": null
  },

  "weapons": {
    "loadout": "all",
    "pickups_enabled": true,
    "start_weapons": ["standard:fps-pistol-9mm", "standard:fps-knife"]
  }
}
```

## Team Deathmatch

```json
{
  "id": "standard:mode-tdm",
  "name": "Team Deathmatch",
  "extends": "standard:mode-deathmatch",
  "team_based": true,
  "teams": { "count": 2, "names": ["Alpha", "Bravo"] },
  "players": { "per_team": 8, "respawn": true, "friendly_fire": false },
  "end_conditions": [
    { "type": "team_score_limit", "frags": 75 },
    { "type": "time_limit", "minutes": 10 }
  ],
  "scoring": { "kill": 1, "suicide": -1, "team_kill": -1 }
}
```

## Search and Destroy (Bomb Defusal)

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

## Capture the Flag

```json
{
  "id": "standard:mode-ctf",
  "name": "Capture the Flag",
  "team_based": true,

  "teams": { "count": 2 },

  "players": {
    "per_team": 8,
    "respawn": true,
    "respawn_delay_sec": 5
  },

  "objectives": {
    "flag": {
      "per_team": 1,
      "capture_requires": "own_flag_at_base",
      "drop_on_death": true,
      "return_timer_sec": 30,
      "carrier_speed_multiplier": 0.85
    }
  },

  "end_conditions": [
    { "type": "captures", "count": 3 },
    { "type": "time_limit", "minutes": 20 }
  ]
}
```

## King of the Hill / Hardpoint

```json
{
  "id": "standard:mode-hardpoint",
  "name": "Hardpoint",
  "team_based": true,

  "teams": { "count": 2 },

  "players": {
    "per_team": 5,
    "respawn": true,
    "respawn_delay_sec": 5
  },

  "objectives": {
    "hardpoint": {
      "zones": ["hp_1", "hp_2", "hp_3", "hp_4", "hp_5"],
      "rotation_time_sec": 60,
      "scoring_rate_per_sec": 1,
      "contested": "no_scoring"
    }
  },

  "end_conditions": [
    { "type": "score_limit", "points": 250 },
    { "type": "time_limit", "minutes": 10 }
  ]
}
```

## Gun Game

```json
{
  "id": "standard:mode-gungame",
  "name": "Gun Game",
  "team_based": false,

  "players": {
    "min": 2,
    "max": 12,
    "respawn": true,
    "respawn_delay_sec": 0
  },

  "weapon_progression": [
    "standard:fps-lmg-negev",
    "standard:fps-shotgun-auto",
    "standard:fps-rifle-ak47",
    "standard:fps-rifle-m4",
    "standard:fps-smg-mp5",
    "standard:fps-sniper-scout",
    "standard:fps-sniper-awp",
    "standard:fps-shotgun-pump",
    "standard:fps-smg-p90",
    "standard:fps-pistol-deagle",
    "standard:fps-pistol-9mm",
    "standard:fps-knife"
  ],

  "rules": {
    "advance_on": "kill",
    "demote_on": "knife_death",
    "demote_amount": 1
  },

  "end_conditions": [
    { "type": "progression_complete", "description": "First player to get a kill with every weapon" }
  ]
}
```

## Arena / Instagib

```json
{
  "id": "standard:mode-instagib",
  "name": "Instagib",
  "team_based": false,

  "players": {
    "min": 2,
    "max": 16,
    "respawn": true,
    "respawn_delay_sec": 1,
    "spawn_protection_sec": 1
  },

  "overrides": {
    "player": {
      "movement": {
        "run_speed": 350,
        "jump_height": 75
      }
    }
  },

  "weapons": {
    "loadout": "fixed",
    "start_weapons": ["standard:fps-railgun-instagib"],
    "pickups_enabled": false
  },

  "end_conditions": [
    { "type": "score_limit", "frags": 30 },
    { "type": "time_limit", "minutes": 10 }
  ]
}
```

## Standard Economy Definitions

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

  "team_elimination_bonus": 0,

  "equipment": {
    "kevlar": { "price": 650 },
    "kevlar_helmet": { "price": 1000 },
    "defuse_kit": { "price": 400, "role": "defenders" }
  }
}
```

## Extending Standard Modes

Games reference a standard mode and override values:

```json
{
  "extends": "standard:mode-search-destroy",
  "name": "Competitive Ranked",

  "rounds": {
    "overtime": {
      "format": "mr6",
      "max_rounds": 12
    }
  },

  "economy": {
    "extends": "standard:economy-competitive",
    "max_money": 12000,
    "kill_rewards_by_category": {
      "sniper": 50
    }
  }
}
```

Only overridden fields change. Everything else inherits from the standard.
