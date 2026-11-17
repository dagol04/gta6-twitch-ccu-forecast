# gta6-twitch-ccu-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/dagol04/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/dagol04/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts GTA VI's launch-day Twitch viewers hour by hour (hours 2–24 of 2026-11-19 UTC) from the hour-1 print alone. GTA VI unlocks at midnight local time country by country, so hour 1 — New Zealand and central Europe live, the UK just unlocking, the US still asleep — is not representative of the day: viewers dip through the small hours, then ramp as the US unlocks and evening arrives in Europe. `main.py` (Python 3.13, standard library only, offline) learns that shape from the case's own comparables: for each past big launch it builds the release-day profile normalised by its hour-1 print, fits a per-hour regression of the profile on log hour-1 viewers (bigger launches have flatter profiles), and calibrates the quantile spread from the dispersion of comparable profiles around the fit — with a separate "late-unlock" regime for launches whose hour 1 was nearly empty because the game had not unlocked yet. No network, no randomness, deterministic.

Validation is leave-one-out over the 37 comparables with full release days in the practice cases (each treated as a target, forecast from its hour-1 print only; the practice cases' own answers are not shipped in the kit — see VALIDATION.md). Mean log-pinball loss: **0.248** on launches with hour-1 ≥ 20k viewers and **0.197** on hour-1 ≥ 50k, against **0.321 / 0.251** for the house persistence baseline; 90% interval coverage 0.86 / 0.91. Typical miss ~37–46% in log terms — launch days are genuinely hard to pin down, and the spread says so honestly.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `<id>.in` in the input directory and writes `<id>.out` (23 rows, hours 2–24, five quantiles each) to the output directory. Each case runs in well under a second.

---

<sub>Entered on [make.fun](https://make.fun/?ref=wb8h7f8k&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
