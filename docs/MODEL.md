# Data model

## Pseudo-random numbers
Every random draw is `u = MOD(SIN(x) * 43758.5453, 1)`, clamped to [1e-6, 1-1e-6], where `x` mixes the day number, the mine number and the seed. The data therefore does not change on recalculation, and changing the seed gives a new data set. Checked offline: uniform distribution, lag correlations near 0, no correlation across seeds.

## Noise and smoothing
Shocks are `z = (u - 0.5) * sqrt(12)` (Uniform) or `NORM.S.INV(u)` (Normal). Smoothing is an AR(1) process around the mean factor mu:

```
f_t = mu + alpha * (f_{t-1} - mu) + K * z_t
K   = sigma * sqrt(1 - alpha^2)   (keep noise amplitude: Yes)
K   = sigma * (1 - alpha)         (keep noise amplitude: No)
f_1 = mu + K / sqrt(1 - alpha^2) * z_1
```
mu and sigma come from the chosen distribution (Uniform: (min+max)/2 and (max-min)/sqrt(12)).

## Trend
Linear: `1 + total * (t-1) / (days-1)`. Exponential: `(1 + total) ^ ((t-1) / (days-1))`.

## Events
An event is a bell curve centred in its window: centre = start + (duration-1)/2, half-width = duration/2, sigma = max(0.2, duration/5), effect = magnitude * exp(-0.5 * ((date-centre)/sigma)^2) inside the window. Which mines it hits:
- **Probability** mode: a mine is hit when its seeded draw is below the event probability.
- **Number of mines** mode: the N active mines with the lowest seeded draws are hit.

## Limits
Up to 20 mines, 8 events and 1100 days. The Parameters sheet shows an error if the range is longer or the end date is before the start date.

## Checks done
Formulas were also recalculated offline with LibreOffice under 11 settings (default, number-of-mines mode, exponential trend, uniform with alpha 0, 1 and 20 mines, short range, 1100 and 1101 days, end before start, events off). No formula errors, and the weekday ratios, trend and event effects match the settings.
