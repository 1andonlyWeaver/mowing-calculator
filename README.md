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

In real-time mode, the app detects when grass breaks spring dormancy before accumulating mowing GDD. Season start requires all five conditions to be met:

1. **Feb 15 biofix** — No GDD accumulates before February 15 (matches MSU GDD Tracker). Universal constant, not species-specific.
2. **Hard freeze reset** — After the biofix, a day with minimum temperature ≤ 25°F (−3.9°C) resets accumulated green-up GDD to zero. Hard freezes kill newly de-hardened tissue, undoing dormancy-break progress. Universal constant.
3. **Day-length floor** — Season cannot start until daylight at the user's latitude exceeds 10.0 hours. This is a latitude-dependent astronomical constraint, not a species-specific biological claim. At 40°N, this is approximately February 3; at 45°N, approximately February 10.
4. **Cumulative GDD threshold** — After the above gates pass, GDD accumulates from the biofix date using a species-specific base temperature. Season starts once the thermal sum reaches the green-up threshold.
5. **Precipitation gate** — A minimum rainfall total over the prior 14 days is required, since thermal accumulation alone cannot induce sustainable green-up without adequate soil moisture.

| Grass type | Green-up base | GDD threshold | Min 14-day precip |
|---|---|---|---|
| Kentucky Bluegrass | 32°F | 200 GDD | 0.25 in |
| Bermuda Grass | 41°F | 150 GDD | 0.50 in |
| Zoysia Grass | 41°F | 175 GDD | 0.40 in |

When the season has not started, the tool shows which condition(s) are still blocking and the current progress toward each threshold.

### Best Day to Mow Recommendation

After calculating the estimated mow date, the app recommends the **best day to mow** within your mow window. It scores each candidate day using four factors:

- **Temperature comfort** (20%) — Ideal mowing conditions are 60–80°F. Days outside this range are penalized proportionally.
- **Precipitation risk** (40%) — Combines forecast probability and expected amount. Rain is the biggest dealbreaker for mowing.
- **Day preference** (30%) — Select your preferred mow days (e.g., Sat + Sun). Preferred days score higher.
- **Proximity** (10%) — Days closer to the agronomically ideal mow date score slightly higher.

**Preferred Mow Days** input (above the Calculate button):
- Check the days of the week you prefer to mow. Defaults to Saturday and Sunday.
- Enable **"Only recommend on these days"** to restrict the recommendation to only those days. If no preferred day falls in the mow window, a fallback message is shown.

### Precipitation Adjustment

Daily MGDD contributions are scaled based on a rolling 7-day precipitation total compared to the grass type's optimal water need. When recent rainfall is below optimal, growth contributions are reduced (down to 50%). A moisture status indicator shows current conditions after calculation.

- **Irrigated toggle**: Enable this to bypass precipitation adjustments if your lawn is irrigated.
- **Climatology mode** does not apply precipitation scaling (30-year averages already reflect typical precipitation).

### Notes

- Internet access is required for map tiles and weather APIs.
- If ACIS or Open-Meteo are temporarily unavailable, try again later.
