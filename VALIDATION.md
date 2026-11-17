# Validation

## Why leave-one-out, not the practice cases directly

The kit ships the four practice cases (p00–p03) but not their answers: each
case file's history ends at the checkpoint (end of hour 1), and the rules
state the answers come from the same historical source (SullyGnome) without
shipping them. Fetching them from SullyGnome directly was tried and failed —
the site is Cloudflare-gated from this machine (HTTP 403) and its game pages
expose 30-day aggregates, not the hourly series. So the practice cases are
scored indirectly: every comparable launch in the kit *does* ship its full
release-day hourly series, so each one can be treated as a target case with
a known answer.

## Procedure (reproducible)

For each of the 37 comparables with a complete 24-hour release day:

1. Build a synthetic case file: `known` = that launch's hour-1 print,
   `comparables` = the other 37 comparables (the target itself excluded),
   `forecast` = hours 2–24.
2. Run the shipped `main.py` on it (`MAKEFUN_INPUT_DIR` / `MAKEFUN_OUTPUT_DIR`).
3. Score the output against the launch's actual hours 2–24 with the
   contest's mean log-pinball loss over all 23 hours x 5 quantiles
   (2 * max(tau*d, (tau-1)*d), d = ln(y) - ln(q)).
4. Do the same for the house persistence baseline (starter code) on the
   same targets.

The scripts used are kept in the build workspace
(`scratch/loo.py`, `scratch/e2e.py` under the bounty folder); the numbers
below are their output on the shipped `main.py`.

## Results (mean log-pinball loss, lower is better)

| Targets | n | This entry | House baseline |
|---|---|---|---|
| All (full release days) | 37 | **0.404** | 0.930 |
| Hour-1 viewers >= 20,000 | 27 | **0.248** | 0.321 |
| Hour-1 viewers >= 50,000 | 20 | **0.197** | 0.251 |
| Hour-1 viewers < 6,000 (late-unlock regime) | 8 | **0.873** | 2.997 |

Median loss on hour-1 >= 20k targets is 0.170 (baseline 0.256). 90%
interval coverage (share of hours inside the 5%–95% band): 0.86 on the
>= 20k set, 0.91 on the >= 50k set (honest target ~0.90). Typical miss
(mean |ln(y) - ln(median)|): ~45% on >= 20k, ~37% on >= 50k.

Per-target losses on the >= 20k set range from 0.094 (ELDEN RING) to 1.112
(Resident Evil Requiem — a launch whose day ramped 12x from a small hour 1;
the baseline scores 1.273 on it). No target in the >= 50k set scores above
0.49 for this entry.

## What was tried and rejected (all on the same LOO harness)

- Median profile only (no scale regression): 0.221 on >= 50k (vs 0.197).
- Empirical residual quantiles for the spread instead of MAD-normal: 0.200
  on >= 50k — no gain, less smooth.
- k-nearest-neighbours in log hour-1 (k = 6/8/12): 0.230–0.235 on >= 50k.
- Theil–Sen robust regression: 0.202 on >= 50k — no gain over OLS.
- Training on all launches (including the tiny hour-1 ones) for the live
  model: 0.285 on >= 50k — the pre-launch hour-1 outliers wreck the fit;
  hence the two-regime split.
- Similarity weights (console-only, rolling-midnight unlock): no gain.
- Viewers-per-channel adjustment: worse both ways; dropped.
- Adding the target's pre-release level (hour-1 vs its 7-day pre-release
  mean) as a second predictor: 0.205 on >= 50k — no gain; dropped.
- Spread multipliers 0.8 / 1.0 / 1.1 / 1.2 / 1.5: 1.1 chosen (best
  >= 20k mean, coverage closest to 0.90).

## Determinism and runtime

`main.py` run twice on the four practice cases writes byte-identical
outputs (verified). Four cases run in ~0.14 s total on this machine — far
inside the 8 s / case limit. No randomness, no wall-clock dependence, no
network; only the standard library.

## Caveats

- LOO over comparables is a proxy: the comparables are past launches of
  many shapes, while the final is a single launch (GTA VI, likely the
  largest ever). The scale regression is clipped to the comparables'
  range, so a record-size hour-1 print saturates at the flattest
  comparable profile rather than extrapolating.
- The practice cases' own scores cannot be computed until their answers
  are published; the LOO numbers above are the honest substitute.
- Practice-case forecasts from the shipped program (for reference):
  p00 Wolverine (hour 1: 23,357) hour-19 median 64,595; p01 RDR2 (116,550)
  hour-19 median 185,808; p02 God of War Ragnarök (269,517) hour-19 median
  322,384; p03 Spider-Man 2 (119,087) hour-19 median 188,457.
