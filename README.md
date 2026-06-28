# RainScore · 雨准

Forecast-vs-actual rainfall scorecard for durian farmers. Compares Open-Meteo
three rain forecast models — **ECMWF IFS 9km** (the WeatherNext baseline), **ECMWF AIFS** (AI), and **JMA GSM** (Japan) — against **ERA5** actual rainfall,
per GPS location, and tells you which model has been more accurate.

Two scores, because farmers care about two different things:
- **Avg miss (MAE)** — how far off the rain *amount* was, in mm/day.
- **Dry/wet calls** — how often the model correctly called rain vs dry day.

## Files
| File | Purpose |
|------|---------|
| `index.html` | The whole app (single file: UI + logic + i18n) |
| `sw.js` | Service worker — offline app shell only |
| `manifest.json` | PWA install metadata |
| `icon-192.png`, `icon-512.png` | App icons |

## Deploy to GitHub Pages
1. Put all five files in the repo root (or a `/docs` folder).
2. Repo → Settings → Pages → Source: your branch, root.
3. Open `https://<user>.github.io/<repo>/` on a phone.
4. Add to Home Screen to install as a PWA.

## Post-deploy test (do this once)
This sandbox can't reach Open-Meteo, so the live data path was **not** tested here —
verify it in a real browser:

1. Open the app, tap **＋** (top right), add a location (type lat/lon or **Use my GPS**).
2. Tap **Update records**.
3. Confirm the table fills with BM / EC forecast numbers and an Actual column.
   - Newest ~5 days show **pending** until ERA5 catches up — that's expected.
4. The verdict card should show MAE and dry/wet % once a few days have actuals.

If forecast numbers stay empty, check the browser console for the Open-Meteo
request and confirm the host isn't blocked by the network.

## How data is captured
- **Forecasts:** logged on every app open, with the last 7 days back-filled from
  Open-Meteo's Previous Runs API (`precipitation_previous_day1` = what was forecast
  ~24h ahead). First capture per day wins — forecasts are never overwritten.
- **Actuals:** ERA5 reanalysis (`precipitation_sum`), ~5-day lag.
- Records grow daily; a full 30-day picture builds over the first weeks.

## Editing a location's GPS
- Move **within 1 km** → comparison history is **kept** (same weather grid cell).
- Move **beyond 1 km** → history resets (different point).
- Threshold is `MOVE_KEEP_M` near the top of the script if you want to tune it.

## Delete / restore
- 🗑 soft-deletes to a recycle bin (records kept; inline **Undo** toast).
- Recycle bin → **Restore** anytime, or **Empty bin permanently** to purge.

## Config knobs (top of the `<script>` in index.html)
- `TZ` — timezone (`Asia/Kuching`).
- `DRY_THRESHOLD` — mm below which a day counts as "dry" (default 1.0).
- `MOVE_KEEP_M` — GPS-edit history-keep radius in metres (default 1000).
- `CACHE_TAG` / `CACHE` in `sw.js` — bump both together on each update so installed
  phones pick up new code.

Data: Open-Meteo (CC BY 4.0). Forecasts: ECMWF IFS 9km (`ecmwf_ifs`), ECMWF AIFS (`ecmwf_aifs025`), JMA GSM (`jma_gsm`); actuals ERA5.

## Models & why three
The purpose is to find a better forecast model for farmers unhappy with the
current WeatherNext broadcast (ECMWF IFS 9km). So the table shows IFS 9km — the
baseline — against two candidate replacements: ECMWF AIFS (ECMWF's own AI model,
clean precipitation in Open-Meteo) and JMA GSM (Japan's global model, strong
heritage for tropical-Asia convection). The verdict highlights the most accurate
model and, when a candidate beats IFS at a location, recommends switching that
farmer.

GraphCast was tried first but Open-Meteo cannot serve its precipitation reliably
(GRIB decode marks it "unknown"), so it returns blank for rain — unusable here.

To change the model line-up, edit the `MODELS` array near the top of the script
(one line per model: `key`, `api` id, `cssVar` colour, `short` label) and add the
matching `models:{...}` labels in both language blocks. Everything else — sync,
scoring, verdict, table — loops over that array automatically.
