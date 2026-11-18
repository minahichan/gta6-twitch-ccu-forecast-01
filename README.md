[![make.fun](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/minahichan/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/minahichan/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/c/gta6-twitch-ccu-forecast-01)

# GTA VI Twitch CCU forecast

Forecasts how many people will be watching Grand Theft Auto VI live on
Twitch in each hour of its release day, 2026-11-19 (UTC), from everything
recorded up to the end of the day's first hour — as five quantiles per
hour (5%, 25%, 50%, 75%, 95%).

## How it works

`main.py` (Python 3, standard library only) reads every case file in the
input directory and writes one forecast file per case. For each case it:

1. Takes the first hour's viewer print as the anchor.
2. Reads the case's comparable past launches and their release-day
   hourly curves.
3. For each hour 2–24, fits a line of the launch's log-ratio
   `ln(viewers_h / viewers_1)` against `ln(viewers_1)` across the
   comparables — bigger first hours mean flatter day curves — and reads
   the line off at the case's own first-hour print. The spread is the
   fit's residual standard deviation, so the five quantiles are
   log-normal around the fitted median.
4. Falls back gracefully (median ratio, then a flat persistence
   forecast) whenever a fit cannot be made, so the output is always
   valid.

The program runs offline, uses no randomness, and writes the same
forecast byte for byte on repeated runs.

## Why the regression works

A launch's day curve is not one shape: it depends on when its audience
can start playing. A console game that unlocks at midnight local time,
country by country, starts its UTC day with only New Zealand, Australia
and the UK in — so a small first hour is followed by a long climb as
Europe and then the Americas unlock (hours 5–6 and 14–21 are the
climbs), while a launch whose first hour is already huge has most of
its audience in from the start and a much flatter day. The log-ratio
regression captures exactly that size dependence, which is why it beats
a single median shape on every practice launch, including the biggest
ones.

## Validation

Scored locally on the kit's four practice launches against their
SullyGnome hourly counts (the same public source the kit used):

| entry | mean pinball loss |
|---|---|
| this forecaster | **0.215** |
| house persistence baseline | 0.387 |

90% interval coverage is about 0.90 on the practice cases. Leave-one-out
validation over the comparable launches agrees (about 0.25 vs 0.28 for
the best non-regression variant).

## Run it

```sh
MAKEFUN_INPUT_DIR=<kit>/input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Writes `out/<id>.out` for each `<id>.in` found, in the format
`{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, ...]}`.

## License

MIT — see [LICENSE](LICENSE).
