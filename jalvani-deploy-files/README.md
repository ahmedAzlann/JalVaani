# 💧 JalVaani — Groundwater Forecasting Dashboard for Maharashtra

> Forecasting groundwater levels 7, 14 and 30 days ahead for 36 Maharashtra districts, and turning the forecasts into plain-language guidance for **farmers** and a priority view for **government officials**.

🔗 **Live demo:** [YOUR-RENDER-URL-HERE](https://YOUR-RENDER-URL-HERE)

![JalVaani start page](docs/screenshots/01-start.png)

## Why this project
Groundwater is the main irrigation source across much of Maharashtra, but readings are published as raw station data that farmers and district officers can't easily act on. JalVaani combines multi-source environmental data with a Temporal Fusion Transformer (TFT) forecast model and presents the results in three languages.

## Features
- **Farmer view** – pick your district and get the water-level outlook and practical tips in simple words, built for phones.
- **Government view** – all 36 districts on one map, ranked priorities, data-gap flags and a CSV export.
- **Forecasts with uncertainty** – t+7 / t+14 / t+30 day predictions with q10 / q50 / q90 ranges.
- **Multilingual** – English, मराठी, हिन्दी and ગુજરાતી, with light and dark themes.
- **Site-guide chatbot** – answers questions using only the numbers shown on screen (LLM via Groq, with an offline fallback guide). Optional voice input and "listen" via Gnani Vachana.
- **Honest by design** – no mock data. Stale readings and passed forecast dates are clearly flagged.

## Screenshots
| Farmer view | Government overview |
|---|---|
| ![Farmer](docs/screenshots/02-farmer.png) | ![Government](docs/screenshots/03-gov-overview.png) |

| Priorities | District detail |
|---|---|
| ![Priorities](docs/screenshots/04-gov-priorities.png) | ![District](docs/screenshots/05-district-detail.png) |

| Marathi | Chatbot |
|---|---|
| ![Marathi](docs/screenshots/06-marathi.png) | ![Chatbot](docs/screenshots/07-chatbot.png) |

## Model results (locked test set, Oct 2025 → Jun 2026)
Error relative to baselines, where **below 1 is better**:

| Horizon | vs "nothing changes" | vs seasonal-naive |
|---|---|---|
| t+7 | 0.841 | 0.940 |
| t+14 | 0.766 | 0.926 |
| t+30 | 0.620 | 0.882 |

All comparisons are statistically significant (p = 0.003). Full methodology: [`docs/REPORT.md`](docs/REPORT.md).

## Tech stack
| Layer | Tools |
|---|---|
| Model | Temporal Fusion Transformer, LightGBM, seasonal-naive ensemble (PyTorch, trained on Colab) |
| Data | CGWB groundwater, CHIRPS rainfall, ERA5-Land, Sentinel-1/2 indices |
| Backend | FastAPI, Uvicorn |
| Frontend | React 18, TypeScript, Vite, Tailwind, Recharts, Leaflet, i18next |
| Hosting | Docker on Render |

## Architecture
The model runs **in batch**, so the API only reads pre-built JSON files. No model server is needed, and every number traces back to a source file.

```
CGWB / CHIRPS / ERA5 data → TFT model (Colab) → build_dashboard_data.py → dev/backend/data/*.json
                                                                              ↓
                                              FastAPI  ←→  React dashboard (en / mr / hi / gu)
```

## Run locally
Requires Python 3.11+ and Node 20+.

```bash
# 1. Backend
python -m pip install -r dev/backend/requirements.txt
cd dev/backend
python -m uvicorn app.main:app --reload --port 8000

# 2. Frontend (new terminal)
cd dev/frontend
npm install
npm run dev          # open http://localhost:5173
```

Optional: copy `dev/backend/.env.example` to `dev/backend/.env` and add `GROQ_API_KEY` (LLM chatbot) and `GNANI_API_KEY` (voice).

## Deploy
The included `Dockerfile` builds the frontend and serves everything from one FastAPI process.

```bash
docker build -t jalvaani .
docker run -p 8000:8000 jalvaani     # http://localhost:8000
```

On **Render**: New → Blueprint → select this repo (it reads `render.yaml`). Add `GROQ_API_KEY` / `GNANI_API_KEY` in the dashboard if you want the LLM chatbot and voice.

## Repository layout
```
dev/                  dashboard app (FastAPI backend + React frontend)
01_data_collection/   data collectors (CGWB, CHIRPS, ERA5, Sentinel)
02_preprocessing/     cleaning and merging
03_models/            baselines (ARIMA, XGBoost, LSTM) and the TFT
04_results/           evaluation outputs and plots
docs/                 report, project status, TFT version history
```

## Acknowledgements
Data from CGWB, CHIRPS and ERA5-Land. Built as a Major Project.
