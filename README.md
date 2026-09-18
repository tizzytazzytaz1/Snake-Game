# Snake Unbound — wagering draft

A single-file HTML5 prototype of a skill-influenced wagering game, reworked from a
free-play arcade snake. **Draft only.** There is no real money, no payment rail and no
cash-out anywhere in this build; stakes and returns are in-game credits held in
`localStorage`.

Open `index.html`, or serve the folder:

```bash
python3 -m http.server 8000
```

## Wager model

Win or loss is decided by **one RNG draw taken at bet time**, at the mode's fixed
`winProbability`. Gameplay never alters that draw — the run is not steered, sabotaged or
assisted. How well the run goes decides only **where inside the payout band** a winning
wager settles.

That split is what makes the math certifiable:

```
RTP(skill) = winProbability × payoutMultiplier(skill)
RTP_min    = winProbability × band.min      hard bound, worst possible play
RTP_max    = winProbability × band.max      hard bound, perfect play
RTP_nominal= winProbability × band.mean     assumes the reference skill model
```

`RTP_min` and `RTP_max` are provable from config constants alone. `RTP_nominal`
additionally assumes the reference skill model in `GAME_CONFIG.skill` and must be
validated against real play data before certification.

| Mode | Win chance | Payout band | RTP min / nominal / max |
| --- | --- | --- | --- |
| Classic | 61.0% | 1.38–1.62× | 84.18% / 91.50% / 98.82% |
| Frenzy Run | 30.5% | 2.76–3.24× | 84.18% / 91.50% / 98.82% |

Frenzy is derived from Classic by a single ratio `FRENZY_RATIO = 0.5`: half the win
chance, double the multiplier. The ratio cancels, so **both modes return identical RTP at
every skill level** and neither can dominate. `assertConfigIntegrity()` checks this at boot
and logs an error if a tuning change ever breaks it.

## Config

Everything tunable lives in `GAME_CONFIG` at the top of the script: credits, bet tiers,
per-mode win probability, payout bands, band score mapping, `JACKPOT_ODDS`,
`JACKPOT_REWARD`, bonus bar, and the RNG settings. Nothing downstream hard-codes a rate or
a multiplier.

## RNG

Every wager-relevant decision goes through `RNG.next()` — one draw per outcome, one draw
per jackpot roll, each tagged in a rolling audit log. The generator is seedable, so any
sequence is reproducible.

`RNG.next()` currently uses **mulberry32**, which is fast and well documented but is not
cryptographically secure and has a 2^32 period. **A certified build must replace that one
function** with an approved hardware or CSPRNG source. Routing every draw through it is
what makes that a one-function change.

## Console helpers

| Call | Does |
| --- | --- |
| `rtpReport()` | Prints published RTP per mode plus the jackpot's stake-dependent contribution |
| `simulateRTP(n, scoreFn)` | Monte-Carlos the live wager model and compares observed to published |
| `forceJackpot()` | Arms the next coin collected to hit |
| `seedRNG(n)` | Reseeds for a reproducible sequence |
| `rngAudit(n)` | Prints recent draws with their tags |

## Known certification gaps

1. **The RNG** is a PRNG, not a certified source. See above.
2. **The jackpot reward is flat**, so its RTP contribution scales inversely with stake:
   about +4% at a 25-credit stake but +20% at 5. Total RTP is therefore stake-dependent,
   which a lab will flag. Making the reward a multiple of stake removes the coupling.
   Left flat because the brief specifies a constant.
3. **`RTP_nominal` rests on a skill assumption** that has not been validated against real
   players.
4. **No server authority.** Balance, outcome and settlement all live client-side in a
   build with no payment rail.
