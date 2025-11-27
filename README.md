# STRAY 🐾

**Animal Feeder with Object Detection and Cloud-Based Monitoring**

_For Mandaluyong Dog Clinic with Crematorium_

[![Demo GIF](docs/images/demo.gif)](docs/images/demo.gif) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](#) [![Tests: Playwright](https://img.shields.io/badge/tests-playwright-blue)](#)

## Overview 📝

**STRAY** is an integrated animal feeding system combining **edge cameras** (ESP32-CAM), **object detection** (YOLO), and a cloud-hosted **web app** for scheduling, monitoring, and auditing feeds. Use the frontend to view live feeds, trigger manual or scheduled dispensing, review gallery images (with detections), and inspect telemetry (weight, battery/solar).

## Hero images / screenshots 📷

- **Dashboard:** `docs/images/screenshot-dashboard.png` — live feed + feed controls
- **Gallery:** `docs/images/screenshot-gallery.png` — saved images with detection overlays
- **Hardware:** `docs/images/hardware-photo.jpg` — feeder hardware and load-cell wiring

## About the project & key objectives ℹ️

**Purpose:** Provide a low-cost, maintainable system for remote or shelter feeders to automate feeding while maintaining visibility and audit trails.

**Key objectives:**

- **Accurate** camera-based detection of animals visiting the feeder
- **Reliable** scheduling and manual feeding controls with cooldowns
- **Telemetry** integration (weight sensor & power monitoring) for dispensing verification and field maintenance
- **Simple developer setup** and clear admin tools for operations

## Features ⚙️

**User (Operator) side:**

- **Live camera preview** (stream + snapshot fallback)
- **Manual feed controls** with presets and custom grams
- **Schedules:** create recurring feeds, timezone-aware
- **Activity Log:** past events, images, and reconciliation status
- **Gallery:** saved snapshots with detection labels and timestamps

**Admin side:**

- **Feeder management:** add/remove feeders, view status
- **User management** and roles
- **Admin Dashboard:** aggregated health, power, and weight-reconciliation metrics
- **Database management** and export tools (see `backend/README.md`)

## Built with 🧰

- **Frontend:** React + TypeScript + Vite + Material UI
- **Backend:** Python Flask + SQLAlchemy (Flask-Migrate used for migrations)
- **Edge device:** ESP32-CAM, optional HX711 + load cell, buck converter for power telemetry
- **Detection:** YOLO (model weights not committed — see notes below)

## Getting Started 🚀

### Quick clone & run (Windows PowerShell)

```powershell
# Clone the repo
git clone https://github.com/<your-org>/stray-feeder.git
cd "feeder-vite-app - Copy"

# Frontend
npm install
npm run dev
# open http://localhost:5173

# Backend (new terminal)
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy env.example .env   # update .env values
python run.py
# API health: http://localhost:5001/api/health
```

### Set up on another device (step-by-step)

Use these steps to set up the project on a different machine (Windows or Linux / Raspberry Pi).

1. Prepare device

   - Ensure **Git** is installed and you can clone repos.
   - If you will run the frontend: install **Node.js v18+**.
   - If you will run the backend: install **Python 3.9+** and ensure `pip` is available.

2. Clone repository

```bash
git clone https://github.com/<your-org>/stray-feeder.git
cd "feeder-vite-app - Copy"
```

3. Install frontend dependencies (optional on low-power devices)

```bash
npm install
# Start in dev mode (for local development):
npm run dev
```

4. Set up backend (recommended on Pi/Ubuntu or server)

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate   # on Linux / macOS
pip install -r requirements.txt
cp env.example .env
# Edit .env (DATABASE_URL, SECRET_KEY, etc.)
python run.py
```

5. (Optional) Install Git LFS for model files

```bash
git lfs install
git lfs track "*.pt"
git add .gitattributes
git commit -m "Track model weights with LFS"
```

6. Verify

- Open frontend: `http://<device-ip>:5173` (if frontend running) or use SSH port-forwarding.
- Check backend: `http://<device-ip>:5001/api/health`.

Notes:

- On constrained devices (Raspberry Pi), prefer running only the backend and/or using pre-built model inference services — avoid running heavy model inference on Pi unless using a lightweight model.

## Firmware & Device Sketches 🔧

The project includes ESP32-CAM firmware sketches used to connect feeder devices to the web platform. You can find the sketches here:

- `backend/YOLO/ESP32-CAM Firmware update/esp32_feeder_quality_update.ino`
- `backend/YOLO/ESP32-CAM Firmware update/esp32_feeder_sketch_reduced_cooldown.ino`

These sketches include device configuration for Wi‑Fi and the backend API endpoint. Before flashing, update the sketch constants (SSID, password, and backend base URL / API key) as described in the top comments of each file.

Quick flashing options:

- Arduino IDE (easy)

```powershell
# 1. Open sketch in Arduino IDE
# 2. Select board: 'AI Thinker ESP32-CAM' (or the matching board)
# 3. Select correct COM port
# 4. Click Upload
```

- PlatformIO (VS Code)

```powershell
# 1. Open the project folder in VS Code
# 2. Use PlatformIO: Import Arduino sketch or create a new platformio project targeting esp32
# 3. Update `platformio.ini` with the correct board and upload port
# 4. Run: pio run --target upload
```

Notes & tips:

- Set the device in boot mode when required (IO0 to GND) for some modules during flashing.
- Confirm camera and motor pins in the sketch match your hardware (servo/motor driver and camera module pins).
- For remote deployments, consider building OTA (over-the-air) firmware updates into the sketch so devices can be updated without physical access.

## Model weights and large files 🔒

YOLO weights (`yolov8n.pt`, `yolov8s.pt`) are large and are not committed. We recommend using **Git LFS** for model files or providing download instructions in `docs/MODEL_DOWNLOAD.md`.

To enable Git LFS for `.pt` files:

```powershell
git lfs install
git lfs track "*.pt"
git add .gitattributes
git commit -m "Track model weights with Git LFS"
```

## Project structure 🗂️

```
backend/           # Flask backend API (models, routes, migrations)
  app.py
  init_db.py
  run.py
  requirements.txt
  models/           # SQLAlchemy models
  routes/           # Flask blueprints for API

src/               # React frontend (Vite + TypeScript)
  api/              # client helpers (auth, gallery, feeders)
  components/       # shared UI components
  pages/            # app pages

public/            # static assets
docs/              # user manual, quickstart, hardware docs, PDF
tests/             # Playwright tests
uploads/           # runtime image uploads (gitignored)
```

## API quick reference 🧾

- `POST /api/auth/login` — **{ username, password }** → access token
- `GET /api/feeders` — list feeders and status
- `POST /api/feeders/:id/trigger` — **{ grams }** trigger a manual feed
- `POST /api/gallery/upload` — base64 image + metadata
- `POST /api/weight-events` — feeder weight telemetry (optional)
- `POST /api/power-status` — power telemetry (optional)

## Testing & CI ✅

- Run Playwright tests:

```powershell
cd tests/playwright
npx playwright test
```

- Consider adding GitHub Actions to run lint/tests on PRs (I can scaffold this for you).

## Contributing 🤝

Please open issues or PRs. For contribution guidelines, add a `CONTRIBUTING.md` if you want a specific workflow. When submitting PRs, **include screenshots** for UI changes and **describe** any required DB migrations.

## License 📜

This project is released under the **MIT License** — add a `LICENSE` file in the repo root.

## Support 💬

For questions or to request help with deployment, open an issue or email the maintainer.

---
