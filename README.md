# Roblox Enemy AI System

A modular server-side enemy AI framework for Roblox featuring pathfinding, line-of-sight detection, melee and ranged combat, parriable projectiles, trigger-based spawning, and proximity-based player health rewards on kill.

> **Note:** The main loop script is a temporary debug handler. Enemy spawning and AI ticking will be managed by a dedicated game state handler in the future.

---

## Features

- **Two combat archetypes** — Short-range melee and long-range projectile attackers
- **Pathfinding with fallback** — Uses `PathfindingService` with a direct `MoveTo` fallback on failure
- **Line-of-sight checks** — Enemies only attack or track players they can actually see
- **Last known position tracking** — Enemies remember where a player was when LoS breaks
- **Surround positioning** — Each enemy uses a unique angular offset to avoid stacking on the same point
- **Parriable projectiles** — Players can reflect enemy projectiles back for double damage
- **Proximity health rewards** — Killing enemies within 100 studs heals the nearest player, scaled exponentially by distance
- **Trigger-based spawning** — `TouchPart` triggers in the workspace fire enemy waves on player contact
- **Active enemy cache** — Main loop uses a flat table to avoid per-frame `FindFirstChild` calls
- **Automatic cleanup** — Dead enemies are removed from the loop and debris-collected after 1 second

---

## Project Structure

```
ServerStorage/
├── SSModules/
│   └── EnemyAI          -- Core AI module (movement, combat, helpers)
└── EnemyModels/
    └── Rig              -- Base enemy rig (cloned on spawn)

ReplicatedStorage/
└── Modules/
    ├── EnemyTypes       -- Enemy stat definitions
    ├── StyleManager     -- Style score integration (Kill, Parry)
    └── PlayerState      -- Shared player state
└── Events/
    ├── PlayerParrying   -- Client → Server parry state
    └── SlamHit          -- Client → Server slam damage

Workspace/
├── Enemies/             -- Active enemy instances are parented here
└── EnemyTriggers/       -- TouchParts that define spawn events (via attributes)
```

---

## Enemy Configuration

Each enemy type is defined in `EnemyTypes` with the following fields:

| Field | Type | Description |
|---|---|---|
| `Speed` | number | `Humanoid.WalkSpeed` |
| `EnemyRange` | string | `"Short"` (melee) or `"Ranged"` (projectile) |
| `Damage` | number | Damage dealt per attack or projectile hit |
| `Range` | number | Attack trigger distance in studs |
| `Hitbox` | number | Melee hitbox size or projectile sphere radius |
| `Cooldown` | number | Seconds between attacks |
| `ProjSpeed` | number | Projectile studs/second (`0` for melee types) |
| `Health` | number | Max and starting health |

### Included Enemy Types

| Enemy | Archetype | HP | Damage | Speed | Range | Notes |
|---|---|---|---|---|---|---|
| Runner | Melee | 50 | 18 | 30 | 1 | Fast, low HP glass cannon |
| Walker | Melee | 100 | 10 | 23 | 5 | Balanced melee unit |
| Tank | Melee | 300 | 30 | 19 | 5 | Slow, high HP bruiser |
| Archer | Ranged | 100 | 50 | 23 | 50 | Fires parriable projectiles |

---

## Spawning Enemies

Enemies spawn from `TouchPart` triggers placed in `workspace.EnemyTriggers`. Each trigger reads three instance attributes:

| Attribute | Type | Description |
|---|---|---|
| `EnemyType` | string | Key into `EnemyTypes` (e.g. `"Archer"`) |
| `Amount` | number | How many enemies to spawn |
| `SpawnLocation` | Vector3 | World position to spawn enemies at |

The trigger fires once on the first confirmed player touch, then destroys itself. Enemies spawn with a small random XZ offset to prevent exact overlap.

---

## AI Behavior

### Short-range (melee)

```
Has line of sight?
├── Yes, within Range  →  Attack + stay still
├── Yes, outside Range →  Pathfind toward player (with surround offset)
└── No                 →  Clear last known position, stop
```

### Ranged

```
Has line of sight AND within Range?  →  Shoot projectile

Not currently pathing:
├── dist < Range × 0.7  →  Retreat to comfortable distance
├── dist > Range + 2    →  Advance toward player
└── otherwise           →  Stay still
```

### Projectile behavior

Projectiles move every `Heartbeat` frame using a `Spherecast` for hit detection.

- If the target player is currently **parrying**, the projectile reverses direction, gains a random angular offset, and its speed increases by 1.5×
- If the reflected projectile then hits the **enemy that fired it**, the enemy takes **double damage**
- Both parry events award `StyleManager.AddStyle("Parry")`

---

## Health Reward on Kill

When an enemy dies, the nearest player within **100 studs** gains health using an exponential proximity multiplier:

```
exponent   = (100 - distance) / 10
multiplier = 0.5 × 1.3^exponent
HP gained  = 10 × multiplier
```

At 0 studs the multiplier is ~13.8× (≈ 138 HP, clamped to 100). At 100 studs the multiplier reaches 0. Players beyond 100 studs receive nothing.

---

## Style System Integration

| Event | Style Action |
|---|---|
| Enemy dies | `"Kill"` (triggers multikill combo in StyleManager) |
| Player parries projectile | `"Parry"` |
| Parried projectile kills enemy | `"Parry"` (second award) |

See the [Weapon System README](../WeaponSystem/README.md) for full style rank and scoring details.

---

## Adding a New Enemy Type

1. Add an entry to `EnemyTypes` with the desired stats.
2. Set `EnemyRange` to `"Short"` or `"Ranged"` — this controls which movement and attack function is used.
3. Place a trigger in `workspace.EnemyTriggers` with the matching `EnemyType` attribute, or call `SpawnEnemy("YourType", position, amount)` directly.

---

## Known Limitations & TODOs

- All enemy types share the same `Rig` model — per-type models via `config.Model` are planned but not yet wired
- `GetClosestPlayer` is duplicated between `EnemyAI` and the main loop script; these should be consolidated
- The main AI loop runs at `task.wait(0.001)` — effectively every frame — which may be aggressive at high enemy counts.
- Last known position is stored as an attribute but is not currently used to resume pathfinding after LoS is regained
- The Archer's `GetOffset` uses `math.random` rather than the deterministic `EnemyID`-based seed that `GetSurroundOffset` uses, so reflected projectiles have no consistent spread pattern, which is intentional
- Slam damage (`SlamHit`) is handled with a flat hardcoded value of 50 rather than reading from a player stat or config

---

## License

MIT — use freely in your own Roblox projects.
