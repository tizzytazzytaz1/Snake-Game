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

`Space` is the only key you have to remember. While you are playing it fires your
ability; everywhere else it means continue — start a run, take a splice, resume, run it
back. `Esc` always backs out.

| Input | Action |
| --- | --- |
| `WASD` / arrow keys | Steer. Inputs are buffered, so fast double-turns register. |
| `Space` | Plasma Breath while playing. Continue on every screen. |
| `Shift` | Amputate 30% of your tail into barricades or turrets. |
| `E` | Shift between Floor 1 and Floor 2. |
| `Q` | Rewind roughly two seconds of time. |
| `P` / `Esc` | Pause. `M` mutes, `H` opens the codex. |
| `1` `2` `3` | Pick a mode on the menu, or a mutation during a splice. |
| `←` `→` | Move the splice cursor, then `Space` to take it. |
| Swipe | Steer on touch devices. Tap to fire. The pads under the board cover the abilities. |

## Gambling Mode (optional)

Off by default. Toggle it on the main menu, or press `G` there. Every prey you eat banks
apples into a wallet (1 each, golden 3, quantum 2) whether the mode is on or off.

With it on, every run-start path opens a bet screen first. Stake apples, and the payout
multiplier grows with both score and survival time. A short run returns less than the stake,
a decent run roughly breaks even, a long one pays well. The stake is deducted up front, so
quitting mid-run forfeits it.

The house has an edge, and it grows. Payout efficiency decays from 100% toward a 60% floor
as you place more lifetime bets, and a rigged-spawn chance climbs from 3% to a 18% cap. A
rigged run drops a wall on you the moment you steer, and nothing saves you from it — not
armor, not Diamond Scales, not Ghost Phase. It says so on the way down.

All of it lives in `GAMBLE_CFG` near the top of the script. `window.gambleDebug()` prints
the current efficiency, trap chance, lifetime return rate and sample multipliers.

With the mode off, none of this code is reachable: no bet screen, no trap roll, and the
cleared spawn runway behaves exactly as it always has.

## Quality of life

- A run is **held at the line** until you steer, so a retry never starts mid-move.
- `Space` on the results screen runs it back immediately, and skips the death animation
  if you press it early.
- The menu remembers your last mode; `Space` starts it.
- Splicing a mutation that binds a key says which key in the notification.
- The control list dims anything you have not unlocked yet.
- The spawn runway is cleared of hazards, so no run opens with lava in your face.
- A run log records what happened and when, so a death is explicable.

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
ticks, so the logic stays deterministic regardless of frame rate.

**Every entity interpolates on its own step clock.** Rivals, mimic spawns and bosses move
at their own cadences, so drawing them against the player's sub-tick progress made them
slide forward, snap back and slide again. Each entity now stamps `lastMoveTick` when it
steps and is drawn across its own interval, linearly — easing here would stall the body at
every cell boundary. Measured over 256 frames, rival motion has zero direction reversals
against 28 under the old scheme.

Static layers (arena gradient, grid, vignette) and all radial glows are pre-rendered into
cached canvases, which keeps a busy frame — boss, turrets, acid and hundreds of particles —
inside the 16.7 ms budget for 60 FPS. CSS `backdrop-filter` is confined to full-screen
overlays; on the small HUD panels it alone cost half the frame budget.

## Interface

The screen, its status bar, the boss meter and the touch pads are one console frame rather
than a stack of floating cards, flanked by two instrument panels that run the full height.
The board is sized from the viewport, so the whole machine fits without scrolling.

Colour is used semantically and sparingly: green for you and your growth, amber for the
combo, red for danger. All iconography is an inline SVG set drawn on a single 24px grid at
one stroke weight — there are no emoji anywhere in the build. Numerals and micro-labels are
set in a monospace face with tabular figures so readouts stay aligned as values change, and
the meters are segmented so they read as instruments rather than progress bars.
