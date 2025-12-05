# STRAY 🐾

**Animal Feeder with Object Detection and Cloud-Based Monitoring**

_For Mandaluyong Dog Clinic with Crematorium_

[![Demo GIF](docs/images/demo.gif)](docs/images/demo.gif) [![License: MIT](https://img.shields.io/badge/license-MIT-green)](#) [![Tests: Playwright](https://img.shields.io/badge/tests-playwright-blue)](#)

## Overview 📝

**STRAY** is an integrated animal feeding system combining **edge cameras** (ESP32-CAM), **object detection** (YOLO), and a cloud-hosted **web app** for scheduling, monitoring, and auditing feeds. Use the frontend to view live feeds, trigger manual or scheduled dispensing, and review gallery images (with detections).

## Images / Screenshots 📷

- **Dashboard:** `docs/images/screenshot-dashboard.png` — live feed + feed controls
- **Gallery:** `docs/images/screenshot-gallery.png` — saved images with detection overlays
- **Hardware:** `docs/images/hardware-photo.jpg` — feeder hardware and load-cell wiring

## About the project & key objectives ℹ️

**Purpose:** Provide a low-cost, maintainable system for remote or shelter feeders to automate feeding while maintaining visibility and audit trails.

**Key objectives:**

- **Accurate** camera-based detection of animals visiting the feeder
- **Reliable** scheduling and manual feeding controls with cooldowns
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
- **Edge device:** ESP32-CAM, optional HX711 + load cell, buck converter for power management
- **Detection:** YOLO (model weights not committed — see notes below)

## Hardware

The project uses the following hardware components (accurate list provided by the maintainer):

- **ESP32-CAM microcontroller** — Responsible for image capture, live video streaming, and controlling the dispensing mechanism. Can be integrated with a model for on-device detection.
- **Servo Motor (SG92R)** — Controls the dispenser lid or mechanism for food dispensing.
- **DC-DC Step-Down Module 5.0 XY-3606 buck converter (HW-688)** — Steps down voltage from a 12V adapter to 5V for the microcontroller and other components.
- **AC/DC 12V 3A Power Adapter** — Supplies power from a wall outlet (12V/3A).
- **Breadboard and Jumper Wires** — For solderless development and temporary connections.
- **DIY Feeder Housing** — Custom-built housing to contain food and protect electronics.
- **NS-08W Solar Panel 8W 9V** — Optional secondary power source for outdoor use.
- **Battery 18650 Li-Ion cell** — Stores power from the solar panel for emergency or night use.
- **Straight Bar Load Cell Weight Sensor 5kg** — Detects total weight of the feed container/bowl.
- **HX711 Load Cell Amplifier Module** — Amplifies the load cell signal and converts it to digital.
- **Bulk Electrolytic Capacitor (400 μF to 1000 μF)** — Energy reservoir to prevent brownouts during current spikes.
- **Decoupling Capacitor 0.1 μF (100nF)** — High-frequency noise filter and local charge reservoir.

## Software requirements

- **Arduino IDE** — For writing and uploading ESP32-CAM sketches.
- **MySQL Workbench** — Visual database design and management tool used during development and testing.
- **XAMPP** — Local Apache + MySQL stack useful for local testing of database-backed features.
- **Visual Studio Code** — Recommended editor for frontend (React + TypeScript) and backend (Flask) development.
- **Machine Learning / Object Detection (YOLOv8)** — YOLOv8 model used for detecting cats and dogs in images/streams.

## Getting Started 🚀

### Quick clone & run (Windows PowerShell)

```powershell
# Clone the repo (your GitHub URL)
git clone https://github.com/heartunderbruhh2/STRAY-Animal-Feeder.git
cd "STRAY-Animal-Feeder"

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
git clone https://github.com/heartunderbruhh2/STRAY-Animal-Feeder.git
cd "STRAY-Animal-Feeder"
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

- Weight/power telemetry available (optional hardware) — see `docs/USER_MANUAL.md` for details.

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

If you plan to host this repository on GitHub (recommended), here are a few quick notes:

- Repository URL: `https://github.com/heartunderbruhh2/STRAY-Animal-Feeder`
- Create the repo on GitHub (web UI) and then push your local branch as shown earlier, or use `gh repo create`.
- If `.pt` files were already committed without LFS, migrate them into LFS to avoid a large repo history:

```bash
# This rewrites history — only use if you understand its impact
git lfs migrate import --include="*.pt"
git push --force origin main
```

Alternatively, remove the large files from history and re-add them tracked by LFS, or store model weights in a release asset or cloud storage and reference the download URL in `docs/`.

## Cross references

- **User manual:** `docs/USER_MANUAL.md`
- **Quick start (printable):** `docs/QUICKSTART.html`
- **Model download & LFS:** `docs/MODEL_DOWNLOAD.md`
- **Hardware tests:** `docs/HARDWARE_TESTS.md`
- **User tests:** `docs/USER_TESTS.md`
- **Admin/backend tests:** `docs/ADMIN_BACKEND_TESTS.md`

**Cross-references (related files)**

- `docs/USER_TESTS.md` — user-focused test plans and checklists.
- `docs/ADMIN_BACKEND_TESTS.md` — backend test plans, DB checks, and admin verification steps.
- `docs/HARDWARE_TESTS.md` — hardware validation procedures (motor tests, installation checks).
- `backend/DATABASE_SETUP.md` — database schema setup and migration notes.
- `docs/QUICKSTART.html` — printable quickstart HTML for operators.
- `docs/QUICKSTART_PDF_INSTRUCTIONS.md` — commands and tips to convert HTML to PDF locally.
- Firmware README & sketches: `backend/YOLO/ESP32-CAM Firmware update/README.md` — flashing instructions and sketch descriptions. See also the `.ino` files in that folder: `backend/YOLO/ESP32-CAM Firmware update/esp32_feeder_sketch_reduced_cooldown.ino` and `backend/YOLO/ESP32-CAM Firmware update/esp32_feeder_quality_update.ino`.
- Root README: `README.md` — project overview and GitHub publishing guidance.

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
- `POST /api/gallery/upload` — base64 image + metadata

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
