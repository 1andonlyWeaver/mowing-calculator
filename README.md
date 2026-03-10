# mowing-calculator

## MGDD Mow Date Estimator

A static web app to estimate the next mow date using Modified Growing Degree Days (MGDD), combining historical temperatures (ACIS PRISM) and a short-term forecast (Open-Meteo GFS).

### New: Climatology View

You can now switch to a climatology-based view that uses PRISM 1991–2020 daily normals (via ACIS GridData) to compute the average number of days it would take to reach your MGDD target for each possible last-mow date (day-of-year). This answers: “On average, if I mow on this date, how many days until I’d expect to mow again?”

How to use:
- Click the mode toggle in section “2. Set Parameters” and choose “Climatology”.
- Pick a location and set Base, Upper, and Target MGDD as usual.
- Click “Calculate Climatology”.
- A chart appears showing Days-to-Target vs the last mow date through the year.

Notes:
- The climatology uses daily normal max/min temperatures aggregated by day-of-year, skipping Feb 29 to create a 365‑day series.
- Growth is computed with the same MGDD method used elsewhere in the app, with your chosen thresholds.

### Quick start

- Option 1: Open `index.html` directly in your browser
- Option 2 (recommended): Serve this folder with a simple static server

```bash
# Python 3
python -m http.server 8000
# then open http://localhost:8000
```

### Usage

1. Select a location on the map (or drag the marker). Latitude/longitude auto-fill.
2. Optionally adjust MGDD parameters: base temp, upper temp, and target accumulation.
3. Set the date you last mowed.
4. Click "Estimate Next Mow Date".

The app will:
- Pull historical daily min/max temperatures and precipitation from ACIS PRISM.
- Pull a 15-day forecast (including precipitation) from Open-Meteo GFS.
- Compute MGDD accumulation since last mowing, adjusted for recent precipitation, and estimate when the target is reached.
- Display a chart and daily breakdown with a Rain column showing daily precipitation.

### Growing Season Detection

In real-time mode, the app detects when grass breaks spring dormancy before accumulating mowing GDD. Season start requires both gates to be satisfied simultaneously:

1. **Cumulative GDD threshold** — GDD accumulates from January 1 using a species-specific base temperature (separate from the mowing base temp). Season starts once the thermal sum reaches the green-up threshold.
2. **Precipitation gate** — A minimum rainfall total over the prior 14 days is required, since thermal accumulation alone cannot induce sustainable green-up without adequate soil moisture.

| Grass type | Green-up base | GDD threshold | Min 14-day precip |
|---|---|---|---|
| Kentucky Bluegrass | 32°F | 200 GDD | 0.25 in |
| Bermuda Grass | 41°F | 150 GDD | 0.50 in |
| Zoysia Grass | 41°F | 175 GDD | 0.40 in |

When the season has not started, the tool shows which gate(s) are still blocking and the current progress toward each threshold.

### Precipitation Adjustment

Daily MGDD contributions are scaled based on a rolling 7-day precipitation total compared to the grass type's optimal water need. When recent rainfall is below optimal, growth contributions are reduced (down to 50%). A moisture status indicator shows current conditions after calculation.

- **Irrigated toggle**: Enable this to bypass precipitation adjustments if your lawn is irrigated.
- **Climatology mode** does not apply precipitation scaling (30-year averages already reflect typical precipitation).

### Notes

- Internet access is required for map tiles and weather APIs.
- If ACIS or Open-Meteo are temporarily unavailable, try again later.
