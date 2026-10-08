# 🌱 SmartAgro

**A multilingual, AI-powered advisory app for Indian farmers: crop disease diagnosis, live mandi prices, weather risk alerts, and crop health insights in one installable web app.**

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Python](https://img.shields.io/badge/Python-3.11-blue.svg)
![Flask](https://img.shields.io/badge/Backend-Flask-black.svg)
![Gemma](https://img.shields.io/badge/AI-Gemma-4285F4.svg)
![PWA](https://img.shields.io/badge/App-PWA-purple.svg)

**Live demo:** https://jis-hackathon-smartagro.onrender.com
**Track:** Open Source + Gemma

---

## Table of Contents

1. [The Problem](#the-problem)
2. [The Solution](#the-solution)
3. [Features in Detail](#features-in-detail)
4. [Why SmartAgro Is Good](#why-smartagro-is-good)
5. [Multi-Model AI Pipeline](#multi-model-ai-pipeline)
6. [Tech Stack and External Services](#tech-stack-and-external-services)
7. [Architecture and Folder Structure](#architecture-and-folder-structure)
8. [Setup and Run Locally](#setup-and-run-locally)
9. [Environment Variables](#environment-variables)
10. [Run with Docker](#run-with-docker)
11. [Deploy on Render](#deploy-on-render)
12. [API Endpoints](#api-endpoints)
13. [How It Works](#how-it-works)
14. [Reliability and Fallbacks](#reliability-and-fallbacks)
15. [Security](#security)
16. [Known Limitations](#known-limitations)
17. [Contributing](#contributing)
18. [License](#license)
19. [Acknowledgements](#acknowledgements)

---

## The Problem

Indian farmers make high-stakes decisions every day, but the information they need is hard to reach:

- **Language barrier.** Most agricultural advice online is in English, while farmers speak many regional languages.
- **Late disease detection.** Spotting a crop disease early often needs an expert who is not nearby, so damage is done before help arrives.
- **Scattered mandi prices.** Prices sit on government portals that are slow, hard to navigate, and sometimes down, and they are quoted per quintal, which is confusing for small sellers who think in kg.
- **Weather risk.** Pests, fungal disease, heat, and rain affect crops, but raw forecasts do not tell a farmer what to do.
- **Limited literacy and connectivity.** Typing long questions or using heavy apps is difficult on low-end phones and weak networks.

## The Solution

SmartAgro puts these tools in one lightweight, installable web app that speaks the farmer's language.

| Problem | SmartAgro Feature | How It Helps |
|---|---|---|
| Information only in English | Multilingual UI, chatbot, and voice | The app and the assistant answer in the farmer's chosen language |
| Late disease detection | AI crop diagnosis from a photo | Upload a leaf photo and get the likely disease and advice in seconds |
| Wrong or irrelevant photos | Image pre-check | Non-crop images are rejected before any AI call, saving time and quota |
| Hard-to-find mandi prices | Mandi Prices page | Live prices in one place, with a quintal/kg toggle |
| Government API downtime | Cached price fallback | The app shows the last saved prices with an update date instead of failing |
| Weather risk is unclear | Alerts and 15-day risk outlook | Forecast data is turned into clear risk levels and seasonal advisories |
| Unclear crop health | NDVI gauge | Shows a crop health score, honestly labelled as Satellite or Estimated |
| Low literacy | Voice input and text-to-speech | Speak a question and hear the answer |
| Poor connectivity and weak phones | Installable PWA | Installs like an app and keeps working with cached content |

---

## Features in Detail

### AI Chatbot (SmartAgro Assistant)
- Answers questions about agriculture, crop disease, irrigation, mandi prices, MSP, Kisan schemes, and how to use the app.
- Replies in the user's chosen language, with localized refusals for off-topic questions so the assistant stays focused on farming.
- Pulls live data (weather, mandi prices) into its answers when a question needs it.
- Powered by **Gemma**, with an automatic Gemini fallback (see [Multi-Model AI Pipeline](#multi-model-ai-pipeline)).

### Crop Disease Diagnosis
- Upload a plant or leaf photo and receive the likely disease, severity, and treatment advice.
- A **pre-check rejects non-crop images** (screenshots, infographics, unrelated photos) without spending an AI call.
- Shows **which model answered** (Gemma, or the fallback) so results are transparent.
- If the primary model fails, the request is retried on a fallback model automatically.

### Mandi Prices
- Live commodity prices sourced from the government's Agmarknet data (data.gov.in).
- **Quintal / kg toggle:** prices are published per quintal, and the toggle converts them (kg = quintal / 100) across cards, table, and charts. The selected unit is remembered between visits and every price is labelled with its unit.
- **Offline-style resilience:** if the government portal is down or slow, the app serves the last saved prices from a bundled cache and shows when they were last updated.

### Weather and 15-Day Risk Outlook
- Current conditions and forecast from weather APIs.
- The Alerts page turns forecast data into a day-by-day risk outlook.
- The page never spins forever: it times out, shows a clear message, and offers a **Retry** button. If a weather provider fails, the last good response is reused.

### Alerts and Seasonal Advisories
- Season-aware advisories for the user's city.
- Humidity-based pest and fungal warnings.
- Optional notifications for important alerts.

### NDVI Crop Health Gauge
- An animated gauge showing crop health.
- **Honest labelling:** the gauge says **Satellite** when the value comes from real satellite data (when `rasterio` and data are available), and **Estimated** when it is a deterministic estimate from real inputs such as recent rain, temperature, season, and location.
- No random values: the same location and date always give the same result, cached per location per day.

### Voice and Accessibility
- Voice input for questions (speech-to-text).
- Text-to-speech for answers, with careful number pronunciation across Indian languages.
- Keyboard handlers, ARIA labels, and landmark tags for assistive technology.

### Settings
- **Language** switching across 10+ Indian languages.
- **Light and dark theme**, with hero sections that follow the active theme.
- **Temperature unit:** Celsius or Fahrenheit, applied wherever temperature is shown.
- **Price unit:** quintal or kg.

### Installable PWA
- Install button, web manifest, and service worker for an app-like experience on phones.

---

## Why SmartAgro Is Good

- **Built for real farmers.** Local languages, voice, simple screens, and a kg toggle for prices.
- **Honest about data.** NDVI is labelled Satellite or Estimated, cached prices show their date, and AI answers show which model produced them. Nothing is invented or random.
- **Resilient.** Every outside service (AI, weather, government prices, satellite data) has a timeout and a fallback, so one failure does not take the app down.
- **Open source.** MIT licensed, with a public repository, clear setup steps, and no secrets in the code.
- **Light and fast.** Flask backend with a vanilla HTML/CSS/JS frontend, with no heavy framework to download on slow networks.
- **Secure by default.** Content Security Policy and security headers, and keys kept in environment variables only.

---

## Multi-Model AI Pipeline

SmartAgro is built around **Gemma** as the primary model for AI features, reached through the Gemini API with `google-genai`.

1. **Primary:** every AI feature (chat, diagnosis, recommendations, alerts, translations) calls the Gemma model set in `GEMMA_MODEL` first.
2. **Fallback:** if Gemma times out, errors, hits quota, or returns unusable output, the request is retried once on **`gemini-3.6-flash`** using the same API key.
3. **Second opinion (diagnosis):** when both models answer, their results are compared. If they agree, the result is marked high confidence. If they disagree, the main answer is shown together with the second model's view and a recommendation to verify with an agricultural expert.
4. **Failure handling:** if both models fail, the user gets a friendly localized message, never a crash.
5. **Transparency:** the diagnosis result shows which model answered.

Voice transcription is the only feature that does not use these models: it uses Groq Whisper.

---

## Tech Stack and External Services

| Area | Technology |
|---|---|
| Backend | Python, Flask |
| Frontend | Vanilla HTML, CSS, JavaScript (PWA) |
| Primary AI | Gemma via the Gemini API (`google-genai`) |
| Fallback AI | Gemini (`gemini-3.6-flash`) |
| Voice to text | Groq Whisper |
| Weather | Visual Crossing, OpenWeather |
| Mandi prices | data.gov.in (Agmarknet) |
| Crop health | `rasterio` satellite data, with a deterministic estimate when unavailable |
| Deployment | Docker, Render |
| License | MIT |

| Service | Used For | Environment Variable |
|---|---|---|
| Gemini API (Gemma and Gemini models) | Chat, diagnosis, recommendations, alerts, translations | `GEMINI_API_KEY`, `GEMMA_MODEL` |
| Groq | Voice transcription | `GROQ_API_KEY` |
| Visual Crossing | Forecasts | `VISUALCROSSING_API_KEY` |
| OpenWeather | Current weather | `OPENWEATHER_API_KEY` |
| data.gov.in | Mandi prices | `DATA_GOV_API_KEY` |

---

## Architecture and Folder Structure

```
SmartAgro-MAIN-2/
├── app.py                      # Flask app: routes, AI pipeline, weather, market, NDVI
├── templates/                  # HTML pages (home, diagnose, market, alerts)
├── static/
│   ├── css/                    # Styles, themes
│   ├── js/                     # Page scripts (alerts.js, market.js, ...)
│   ├── manifest.json           # PWA manifest
│   └── sw.js                   # Service worker
├── market_history_cache.json   # Seed data for the mandi price fallback
├── chat_city_aliases.json      # City name aliases for the chatbot
├── requirements.txt            # Python dependencies
├── Dockerfile                  # Container build
├── runtime.txt                 # Python version for hosting
├── .env.example                # Template for environment variables
├── .gitignore                  # Keeps .env and runtime files out of git
├── LICENSE                     # MIT license
└── README.md
```

Runtime files such as usage counters and NDVI caches are created while the app runs and are not committed.

---

## Setup and Run Locally

**Prerequisites:** Python 3.11 (recommended), `pip`, and API keys for the services listed above.

```bash
# 1. Clone
git clone https://github.com/Anant-083/JIS_Hackathon-Smartagro-.git
cd JIS_Hackathon-Smartagro-

# 2. Create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
cp .env.example .env            # then edit .env and add your keys

# 5. Run
python app.py
```

Open the printed local address in your browser.

> **Python 3.14 note:** `rasterio` has no ready-made build for Python 3.14. If it fails to install, remove it from your local install. The app still starts and shows an **Estimated** NDVI. Python 3.11 is recommended.

---

## Environment Variables

Copy `.env.example` to `.env` and fill in your own values. **Never commit `.env`.**

| Variable | Required | Purpose |
|---|---|---|
| `GEMINI_API_KEY` | Yes | Key for the Gemini API, used for Gemma and the Gemini fallback |
| `GEMMA_MODEL` | Optional | Gemma model name (a default is used if empty) |
| `GROQ_API_KEY` | Optional | Voice transcription |
| `VISUALCROSSING_API_KEY` | Recommended | Forecasts and the 15-day outlook |
| `OPENWEATHER_API_KEY` | Recommended | Current weather |
| `DATA_GOV_API_KEY` | Recommended | Live mandi prices |
| `FLASK_DEBUG` | Optional | `1` for local development, `0` in production |

Without optional keys, the related feature falls back or shows a clear message, and the rest of the app keeps working.

---

## Run with Docker

```bash
docker build -t smartagro .
docker run -p 7860:7860 --env-file .env smartagro
```

Then open `http://localhost:7860`.

---

## Deploy on Render

1. Push the repository to GitHub (make sure `.env` is **not** included).
2. On [render.com](https://render.com), choose **New → Web Service** and connect the repository.
3. Choose the **Docker** runtime (the repository includes a `Dockerfile`).
4. Open the **Environment** tab and add each variable from the table above, using your own keys. Set `FLASK_DEBUG=0`.
5. Deploy. When the status shows **Live**, open the URL and test chat, diagnosis, Mandi Prices, and Alerts.

> On Render's free tier the service sleeps when idle, so the first request after a pause can take a minute.

---

## API Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/` | Home page |
| POST | `/api/chat` | Chatbot (Gemma first, Gemini fallback) |
| POST | `/api/diagnose` | Crop disease diagnosis from an image |
| POST | `/api/seasonal-alerts` | Seasonal advisories for a city |
| GET | `/healthz` | Health check |

Other routes cover weather, the 15-day outlook, mandi prices, NDVI, and voice transcription. See `app.py` for the complete list.

---

## How It Works

**Chat flow**
1. The user sends a question in their language.
2. The server checks that the topic is about farming, otherwise it replies with a localized refusal.
3. If the question needs live data (weather, mandi prices), the server fetches it and passes it to the model.
4. Gemma generates the answer. If it fails, the Gemini fallback answers.
5. The reply is shown (and can be read aloud).

**Diagnosis flow**
1. The user uploads a photo.
2. A pre-check rejects non-crop images without any model call.
3. The image goes to Gemma, with the fallback if it fails.
4. The response is validated, then shown with the model name and a confidence note.

**Market flow**
1. The server requests prices from data.gov.in with a short timeout and one retry.
2. On success, prices are shown and the cache is refreshed.
3. On failure or empty data, the saved cache is shown with its update date.
4. The quintal/kg toggle converts the displayed prices in the browser and remembers the choice.

---

## Reliability and Fallbacks

| Failure | What the User Sees |
|---|---|
| Gemma fails | The answer comes from the Gemini fallback, and the model label shows it |
| Both AI models fail | A friendly localized message |
| Government price API down | Last saved prices with an update date |
| Weather provider fails | The last successful response, or a clear message with a Retry button |
| Satellite data missing | NDVI marked **Estimated** |
| No network | The installed PWA still opens cached pages |

---

## Security

- API keys live only in environment variables, never in the code or the repository.
- `.env` is listed in `.gitignore`, and `.env.example` contains only placeholders.
- Content Security Policy and other security headers are set on responses.
- Voice and image inputs are validated before use.
- Rotate any key that has ever been shared or shown in a screenshot.

---

## Known Limitations

- Government data (Agmarknet) can be slow or down, so cached prices may be older than live prices.
- Without satellite data or `rasterio`, NDVI is an estimate, and it is labelled that way.
- `rasterio` does not install on Python 3.14, so use Python 3.11.
- Render's free tier has cold starts, and its disk resets on restart, so runtime caches are rebuilt.
- AI diagnosis is general guidance and not a replacement for an agricultural expert. Always verify serious cases locally.
- Gemma and Gemini quotas depend on your Google project, and heavy use can hit limits.

---

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a branch: `git checkout -b feature/your-feature`.
3. Make your changes and test them locally.
4. Never commit `.env` or any secret.
5. Open a pull request describing what you changed and why.

---

## License

Released under the [MIT License](LICENSE).

---

## Acknowledgements

- Google's **Gemma** and **Gemini** models, through the Gemini API
- **Groq** for Whisper speech-to-text
- **Visual Crossing** and **OpenWeather** for weather data
- **data.gov.in / Agmarknet** for mandi price data
- The open-source Python and Flask community
- Everyone who tested the app and gave feedback, including collaborators on earlier versions

---

*Made for Indian farmers. If it helps even one farmer decide better, it did its job.* 🌾
