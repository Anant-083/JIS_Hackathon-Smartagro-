---
title: Smartagro
emoji: 🌱
colorFrom: green
colorTo: blue
sdk: docker
app_port: 7860
pinned: false
---

# SmartAgro

An open source farming assistant for Indian farmers, built with Flask and Google Gemma.

[![MIT License](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-3.0-black.svg)](https://flask.palletsprojects.com/)
[![Gemma](https://img.shields.io/badge/AI-Gemma-4285F4.svg)](https://ai.google.dev/gemma)

**Live demo:** [SmartAgro on Hugging Face Spaces](https://alphacoder7206-smartagro.hf.space/)

## The problem

Farmers can have difficulty finding farming information in their preferred language, getting quick help when a crop looks diseased, comparing mandi prices, and preparing for weather risks. Limited literacy, mobile data, and unreliable connectivity make long or scattered information harder to use.

## The solution

| Problem | Feature | How it helps |
|---|---|---|
| Farming information is often hard to access in a preferred language | Multilingual interface and Kisan Helper | Translates page content and lets farmers ask farming questions in supported languages. |
| Crop disease identification takes time | Crop diagnosis | Accepts a crop image, checks whether it appears to show plant material, then asks the shared Gemma and Gemini fallback helper for an analysis. |
| Mandi rates are scattered | Mandi prices | Displays government Agmarknet observations by supported city, with saved price history as a fallback. |
| Weather can damage crops | Weather and alerts | Shows current conditions, forecast data, and rule or AI generated crop risk guidance. |
| Literacy and connectivity can be limited | Voice controls and PWA caching | Supports speech input and spoken replies; previously visited pages and static files can be available offline. |

## Features

### AI chatbot

All model-backed features call one shared generation helper. It tries the configured Gemma model first (`gemma-4-26b-a4b-it` by default, configurable with `GEMMA_MODEL`) and then tries `gemini-3.6-flash` if Gemma errors or returns no text. Both use `GEMINI_API_KEY`; if both calls fail, the feature returns its own fallback or error. The fallback request uses the Gemini GenerateContent API with supported generation fields and omits Gemma's `thinking_config`. Chat history is trimmed so its final turn is always a user turn. Off-topic questions are refused before any model call using localized canned replies.

### Crop disease diagnosis

Upload an image for an AI generated crop health assessment. The backend validates the base64 payload and size, then asks the shared AI helper to classify whether it shows plant material before generating two diagnosis passes. The response identifies the model used for each pass. If the classifier call itself fails at both model providers, the current code allows diagnosis to continue and the later vision pass may still fail. No live model provider response has been verified in this checkout.

### Mandi prices

Prices are fetched from data.gov.in Agmarknet data. The interface can show rupees per quintal or per kilogram; kilogram values are quintal values divided by 100 and shown to two decimal places. The saved preference applies to price cards, ticker, table, and charts. Live min/max/modal values are displayed when returned by Agmarknet; history fallback contains saved modal prices and may not have min/max values. If the live feed is empty or unavailable, persisted market history is used where present; no price is invented when neither source has one.

### Weather and 15-day risk outlook

OpenWeatherMap provides current weather and its forecast. Visual Crossing can extend the forecast when configured. Only returned provider days are presented as forecast data; days without data are marked unavailable. The Alerts outlook has a timeout and a Retry control.

### Alerts and notifications

The Alerts page shows weather and crop risk information, a forecast outlook, and seasonal advice. Browser notification permission and daily reminder time can be set in Settings. Notification delivery depends on browser support and permission; the app does not provide a server push subscription service.

### Crop health gauge

The gauge is explicitly labeled **Satellite**, **Estimated**, or **Unavailable**. When `rasterio` is installed and a usable Sentinel-2 scene can be read, it calculates NDVI from red and near-infrared imagery served through Earth Search STAC. If imagery cannot be retrieved, it returns a deterministic estimate from the rounded location, date, current temperature, and rainfall, cached per location and UTC day. The estimate is a heuristic, not measured NDVI, and is labeled **Estimated**.

### Voice, language, settings, and install

Kisan Helper supports browser speech synthesis and browser speech recognition; recorded audio can also be sent to Groq Whisper for transcription. Language selection translates page content and sets the chatbot response language. Settings include light/dark/system themes, Celsius/Fahrenheit, quintal/kg, notification controls, text-to-speech and voice-input toggles, and voice volume. Preferences persist in local storage. The manifest and service worker provide install metadata and cache visited pages and static assets for offline use in compatible browsers. Browser permission, speech-language availability, and install prompts vary by browser.

## Why it is useful

- **Accessible:** language, voice, font size, and unit settings are available in the interface.
- **Offline resilience:** the service worker caches pages and assets, while market history is persisted on the server.
- **Honest labels:** estimated NDVI and missing forecast or market data are identified instead of shown as observations.
- **Security basics:** responses include a Content Security Policy and common browser security headers; `.env` is ignored by Git.
- **Open source:** released under the MIT License.

## Technology and external services

| Service or library | Used for | Environment variable |
|---|---|---|
| Google Gemini API | Gemma first, then `gemini-3.6-flash` fallback for chat, image diagnosis, crop recommendations, selected alerts, and translations | `GEMINI_API_KEY`; optional Gemma model override `GEMMA_MODEL` |
| Groq Whisper | Chatbot speech-to-text | `GROQ_API_KEY` |
| OpenWeatherMap | Current weather and short forecast | `OPENWEATHER_API_KEY` |
| Visual Crossing | Extended daily forecast | `VISUALCROSSING_API_KEY` |
| data.gov.in Agmarknet | Mandi price observations | `DATA_GOV_API_KEY` |
| Element84 Earth Search STAC and Sentinel-2 COGs | Satellite imagery for NDVI | No key; `rasterio` and `numpy` must be installed |
| Open-Meteo geocoding | City lookup for chatbot live-data requests | No key |
| Flask, Gunicorn, requests, python-dotenv | Web server, HTTP calls, and configuration | — |

## Architecture and files

```text
.
├── app.py                         Flask routes, provider calls, caches, and server logic
├── chat_city_aliases.json          City names used by chatbot live-data lookup
├── market_history_cache.json       Persisted market price history
├── requirements.txt                Python dependencies
├── runtime.txt                     Python runtime declaration
├── Dockerfile                      Container build and Gunicorn command
├── LICENSE                         MIT license text
├── .env.example                    Environment variable template
├── README.md                       Project documentation
├── templates/
│   ├── index.html                  Home dashboard
│   ├── diagnose.html               Crop diagnosis page
│   ├── market.html                 Mandi prices page
│   ├── alerts.html                 Weather and crop alerts page
│   ├── usage.html                  Usage counters page
│   └── offline.html                Offline navigation fallback
└── static/
    ├── css/                        Page and shared stylesheets
    ├── js/
    │   ├── main.js                 Shared navigation, install, and notification logic
    │   ├── dashboard.js            Weather, crop, map, and vegetation UI
    │   ├── diagnose.js             Image upload and diagnosis UI
    │   ├── market.js               Prices, unit display, filters, and charts
    │   ├── market_translate.js     Market translation helpers
    │   ├── alerts.js               Alerts, forecast outlook, and translations
    │   ├── kisan-helper.js         Chat, voice, and text-to-speech UI
    │   ├── settings.js              Persistent user settings
    │   ├── profile.js               Profile controls
    │   └── translations.js         Browser interface translations
    ├── icons/                      PWA icons
    ├── manifest.json               PWA metadata
    └── service-worker.js           Offline page and asset caching
```

## Setup

### Prerequisites

Python 3.11 is the supported runtime. Docker builds install GDAL libraries for `rasterio`. For local non-Docker installation, rasterio may need compatible GDAL system libraries; without rasterio, the gauge falls back to an estimate.

### Install and run

```bash
git clone https://github.com/Anant-083/Smartagro-Main.git
cd Smartagro-Main
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Edit `.env` with your own provider keys. Never commit it. Then run:

```bash
python app.py
```

`python app.py` listens on `PORT` (default `7860`). For production-like local serving:

```bash
PORT=7860 gunicorn --bind 0.0.0.0:7860 --workers 1 --threads 8 --timeout 60 app:app
```

### Environment variables

| Variable | Purpose | Required |
|---|---|---|
| `GEMINI_API_KEY` | Gemini API access for Gemma features | For AI features |
| `GEMMA_MODEL` | Gemma model identifier; defaults in `app.py` | No |
| `OPENWEATHER_API_KEY` | Current weather and forecast | For live weather |
| `VISUALCROSSING_API_KEY` | Extended forecast days | No; outlook may be shorter |
| `GROQ_API_KEY` | Whisper voice transcription | No; voice transcription unavailable without it |
| `DATA_GOV_API_KEY` | Agmarknet feed access | For current mandi data |
| `PORT` | HTTP listen port; defaults to `7860` | No |
| `FLASK_DEBUG` | Enables gated debug endpoints when set to `1` | No; keep `0` in deployment |
| `LOG_LEVEL` | Python logging level | No |
| `DIAGNOSIS_LOG_DIR` | Local diagnosis audit log directory | No |

### Docker

```bash
docker build -t smartagro .
docker run --rm -p 7860:7860 --env-file .env smartagro
```

## Deploy on Render

1. Push this repository to a Git provider and create a new **Web Service** in Render from that repository.
2. Select Docker as the runtime so the included `Dockerfile` installs GDAL and binds to Render's `PORT` value (with `7860` as the local and Hugging Face default).
3. In the service dashboard, open **Environment** and add the variables from the table above. Set `FLASK_DEBUG` to `0`; never put credentials in source code or README files.
4. Deploy and wait for the service to become healthy. Check `/healthz` for process health and `/readyz` for configured provider indicators.
5. Add or rotate provider credentials in Render’s dashboard when needed, then redeploy.

Render free instances may sleep when idle. Persistent files such as market history and NDVI cache require a persistent disk if they must survive instance replacement; without one, the app can still run but regenerates those caches.

## API endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | Home dashboard |
| GET | `/diagnose` | Diagnosis page |
| GET | `/market` | Mandi page |
| GET | `/alerts` | Alerts page |
| GET | `/offline` | Offline fallback page |
| GET | `/usage` | Usage dashboard |
| GET | `/healthz`, `/readyz` | Health and configuration status |
| GET | `/api/weather` | Current weather and merged forecast |
| GET | `/api/vegetation` | Satellite or estimated vegetation index |
| GET | `/api/market` | Agmarknet market data and saved fallback |
| POST | `/api/chat` | Kisan Helper chat |
| POST | `/api/stt` | Groq Whisper speech transcription |
| POST | `/api/diagnose` | Crop image diagnosis |
| GET | `/api/diagnose-log`, `/api/diagnose-log/image/<path:filename>`, `/api/diagnose-log/accuracy` | Diagnosis review data, debug mode only |
| POST | `/api/diagnose-log/review` | Save human review, debug mode only |
| POST | `/api/crop-recommendations` | Crop suggestions |
| POST | `/api/alerts` | Current weather alerts |
| POST | `/api/alerts-forecast` | Forecast-day alerts |
| POST | `/api/monthly-alerts` | Risk outlook for available forecast days |
| POST | `/api/seasonal-alerts` | Seasonal advice |
| POST | `/api/crop-risk` | Crop risk analysis |
| POST | `/api/translate-market`, `/api/translate-alerts`, `/api/translate-dashboard`, `/api/translate-diagnose`, `/api/translate-diagnosis-result` | Translate page content |
| POST | `/api/translate-market/clear` | Clear market translation cache |
| GET | `/api/debug-market`, `/api/debug-extended-forecast` | Provider diagnostics, only when `FLASK_DEBUG=1` |
| GET | `/api/usage` | Read usage counters |
| POST | `/api/usage/reset` | Reset usage counters |

Debug and diagnosis review routes are gated by `FLASK_DEBUG=1`; leave debug mode off in deployment.

## How it works

**Chat:** the browser sends the conversation, language, temperature preference, and available weather context. The server detects supported live-data intents, fetches relevant data, and supplies that context to the shared Gemma then Gemini fallback helper. Off-topic requests are refused before the model call.

**Diagnosis:** the browser posts a validated image payload. The server checks size and base64 decoding, asks the shared helper whether it is plant material, then generates and combines diagnosis analysis. Provider and validation errors are returned to the page.

**Market fallback:** the server requests Agmarknet observations by state and saves price history. If the live feed has no usable data, the most recent saved genuine observations are used where available; otherwise the city is shown without fabricated prices.

## Known limitations

- Render free-tier services can cold start after idle periods.
- Government Agmarknet data and its API can be delayed or unavailable; cached history can also be absent on ephemeral storage.
- NDVI is estimated when `rasterio`, imagery, or a readable scene is unavailable. Estimated values are not satellite measurements.
- The included runtime and Docker base pin Python 3.11.9. This checkout was run with Python 3.14.2; a Docker build and Python 3.11.9 runtime were not run here.
- Weather and extended forecast coverage depends on valid provider credentials and provider availability. The outlook only reports returned forecast days.
- Browser speech recognition and installation prompts vary by browser. Whisper transcription needs `GROQ_API_KEY`.
- This environment did not have rasterio installed, and live provider responses were not verified as part of repository checks.

## Contributing

Issues and pull requests are welcome. Keep secrets in environment variables, preserve honest labels for unavailable or estimated data, and describe any checks performed with a change.

## License and acknowledgements

This project is licensed under the [MIT License](./LICENSE). Thanks to the open source maintainers behind Flask, Rasterio, NumPy, Leaflet, and Chart.js, and to the providers of Gemma, weather, market, and satellite data services.
