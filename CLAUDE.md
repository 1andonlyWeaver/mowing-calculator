# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MGDD Mow Date Estimator — a static web app that estimates the next lawn mowing date using Modified Growing Degree Days (MGDD). The entire application lives in a single `index.html` file with no build system, bundler, or package manager.

## Development

Serve locally with any static file server:
```bash
python -m http.server 8000
# Open http://localhost:8000
```

Or open `index.html` directly in a browser. No build step, no dependencies to install.

## Architecture

**Single-file app** (`index.html`, ~2200 lines): HTML + Tailwind CSS + vanilla JavaScript, all inline. No modules, no separate JS/CSS files.

### External Dependencies (loaded via CDN)

- **Tailwind CSS** — utility-first styling
- **Leaflet** + **Leaflet Control Geocoder** — interactive map and address search
- **Chart.js** + **chartjs-plugin-annotation** — accumulation plots and target threshold line

### Two Operating Modes

1. **Real-time mode**: Fetches historical temps from ACIS PRISM and a 15-day forecast from Open-Meteo GFS, computes MGDD accumulation since last mow, and estimates the next mow date.
2. **Climatology mode**: Fetches 30 years of PRISM data (1991–2020), computes daily normal MGDD, and shows average mowing interval by day-of-year plus monthly median bar chart.

### Key Functions

- `calculateGDD(min, max, base, upper)` — core MGDD formula for a single day
- `getPrecipScaleFactor(precipWindow, optimalPrecip)` — scales daily MGDD by 7-day rolling rainfall vs grass optimal need (0.5–1.0 multiplier; bypassed in irrigated mode)
- `computeClimateNormals(normalsData, baseT, upperT)` — builds DOY-keyed average GDD from 30-year ACIS data for post-forecast extension
- `findGrowingSeasonStart(times, mins, maxs, precips, grassType, baseTemp, lat)` — 5-condition dormancy-break detector (Feb 15 biofix, cumulative green-up GDD, hard freeze reset, day-length floor, precipitation)
- `diagnoseSeasonStatus(...)` — returns human-readable growing season status for the info banner
- `fetchAcisData(lat, lon, start, end)` — historical weather from ACIS GridData (PRISM grid 21, POST to `data.rcc-acis.org/GridData`)
- `fetchOpenMeteoData(lat, lon, start, end)` — forecast weather from Open-Meteo GFS; fetches `precipitation_probability_max` alongside temps and precip
- `handleCalculation()` — real-time mode pipeline: fetch data, accumulate MGDD, find crossing date, render chart/table
- `handleClimatologyCalculation()` — climatology mode pipeline: fetch 30-year normals, average by DOY, compute days-to-target for each start DOY
- `renderForecastTable(rows, targetGDD)` — renders day-by-day forecast/normals table including precipProb column
- `renderWeatherStrip(strips)` — renders compact weather icon strip above the chart
- `renderMoistureStatus(data)` — renders soil moisture / precipitation status card
- `renderGddChart()` / `renderClimoChart()` / `renderClimoMonthlyChart()` — Chart.js rendering

### Data Flow

Both ACIS and Open-Meteo responses are normalized to a common shape: `{ daily: { time[], temperature_2m_min[], temperature_2m_max[] } }`. ACIS returns data in `[date, maxt, mint]` rows with `'M'` for missing values.

### UI Patterns

- Grass type presets (Kentucky Bluegrass, Bermuda, Zoysia) auto-fill base/upper/target but revert to "Custom" on manual edit
- Geolocation with IP-based fallback chain (ipapi.co → ipwho.is)
- `waitForNextFrame()` yields to the browser during heavy computation loops to keep spinners animated
- Mobile: auto-scrolls to results after calculation on viewports < 1024px
- Unit system toggle (metric/imperial): `applyUnitToggle()` converts displayed °F/in values on-the-fly; all internal calculations and storage remain in imperial
- Climate normals toggle (`#climateNormalsToggle`): when checked, extends real-time mode predictions beyond the 15-day Open-Meteo window using `computeClimateNormals()`; extended rows have `isNormals: true` which suppresses precipProb display
- Precipitation probability: fetched as `precipitation_probability_max` from Open-Meteo GFS, stored as `precipProb` on each forecast row, shown in the forecast table and weather strip; hidden for normals rows and suppressed in irrigated mode

## Documentation Maintenance

When making changes that appreciably affect how the tool works — new features, changed behavior, modified data sources, new grass presets, updated thresholds, or changes to the UI flow — update both:

1. **`README.md`** — keep feature descriptions, data source details, and usage instructions current.
2. **About modal in `index.html`** (`#about-overlay`) — keep the user-facing explanations in sync with actual behavior. Sections most likely to need updates: "What is this tool?", "Two modes", "Grass type presets", "Data sources", and "Growing Season Detection".

Minor changes (bug fixes, styling tweaks, refactors with no behavior change) do not require documentation updates.

## Git Workflow

Uses **git-flow**: `main` for releases, `develop` for integration, `feature/*` and `release/*` branches. Tags mark versions (e.g., `0.1.0`, `0.1.1`).
