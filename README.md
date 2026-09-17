# Snake Unbound: Ultra Evolution

A single-file, dependency-free HTML5 arcade rogue-lite. Open `index.html` in any modern
browser — there is no build step, no package manager, and no external asset of any kind.
Graphics are drawn with the Canvas 2D API, every sound is synthesised live through the Web
Audio API, and all progress is kept in `localStorage`.

## Play

Open `index.html`, or serve the folder and visit it:

```bash
python3 -m http.server 8000   # then open http://127.0.0.1:8000
```

## Modes

| Mode | What changes |
| --- | --- |
| Classic Evolution | The traditional snake loop with the in-run DNA mutation system layered on top. |
| Frenzy Run | Faster ticks, denser spawns, and up to three rival AI snakes competing for food. |
| Dimensional Shift | Two stacked sub-grids. Shift floors to escape dead ends and your own coils. |

## Controls

| Input | Action |
| --- | --- |
| `WASD` / arrow keys | Steer. Inputs are buffered, so fast double-turns register. |
| `Space` | Plasma Breath, once the mutation is spliced. |
| `Shift` | Amputate 30% of your tail into barricades or turrets. |
| `E` | Shift between Floor 1 and Floor 2. |
| `Q` | Rewind roughly two seconds of time. |
| `P` / `Esc` | Pause. `M` mutes, `H` opens the codex. |
| `1` `2` `3` | Pick a mutation during a DNA splice. |
| Swipe | Steer on touch devices. Tap the board to fire. On-screen buttons cover the abilities. |

## The DNA engine

Biomass grants XP. Each level triggers a splice that offers three of ten mutations, and
most stack to rank 2 or 3.

Hydra Head · Diamond Scales · Plasma Breath · Acid Trail · Gravity Well · Tail Turret ·
Chronos Core · Ghost Phase · Metabolic Surge · Venom Fangs

## Prey, rivals and bosses

Scarab fruit flees your head. Mimic berries hatch hostile spawns when eaten. Golden
clusters merge with nearby food into fatter targets. Quantum fruit splits you into a
mirrored twin until you collect the collapse shard. Rival snakes path with A* and compete
for the same food you want.

Every 100 points wakes a boss. **The Grid-Eater** crawls the perimeter and converts it to
lava, shrinking the arena until you kill it and win the space back. **The Hydra Centipede**
hunts you down and splits in two whenever you strike a glowing weak spot.

Floor hazards round it out: frictionless ice, directional conveyors, and lava vents that
acid can dissolve.

## Architecture

Everything lives in `index.html` under clearly separated subsystems: `SoundEngine`,
`ParticleSystem`, `World`, `SnakeEntity`, `FoodManager`, `MutationEngine`, `AIManager`,
`Boss`, `Game`, `Renderer`, `UI` and `Input`.

The simulation runs on a fixed-timestep accumulator while rendering interpolates between
ticks, so movement stays smooth and the logic stays deterministic regardless of frame rate.
Static layers (arena gradient, grid, vignette) and all radial glows are pre-rendered into
cached canvases, which keeps a busy frame — boss, turrets, acid and hundreds of particles —
inside the 16.7 ms budget for 60 FPS.
