# mowing-calculator

## MGDD Mow Date Estimator

A static web app to estimate the next mow date using Modified Growing Degree Days (MGDD), combining historical temperatures (ACIS PRISM) and a short-term forecast (Open-Meteo GFS).

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
- Pull historical daily min/max temperatures from ACIS PRISM.
- Pull a 15-day forecast from Open-Meteo GFS.
- Compute MGDD accumulation since last mowing and estimate when the target is reached.
- Display a chart and daily breakdown.

### Notes

- Internet access is required for map tiles and weather APIs.
- If ACIS or Open-Meteo are temporarily unavailable, try again later.
