# 🌊 DeepDrishti — AI-Powered Marine Surveillance System

> **DeepDrishti** ("Deep Vision" in Sanskrit) is a full-stack, real-time marine surveillance platform that combines deep learning image enhancement, YOLOv8-based underwater object detection, and MiDaS depth estimation, all served through a Node.js/Express REST API and a React dashboard.

---

## 📑 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [AI / ML Pipeline (model/)](#ai--ml-pipeline-model)
- [Backend API (backend/)](#backend-api-backend)
- [Frontend Dashboard (frontend/)](#frontend-dashboard-frontend)
- [Database Schema](#database-schema)
- [RBAC — Roles & Permissions](#rbac--roles--permissions)
- [API Reference](#api-reference)
- [Configuration & Environment Variables](#configuration--environment-variables)
- [Getting Started](#getting-started)
- [Seeding the Database](#seeding-the-database)
- [Default Login Credentials](#default-login-credentials)
- [Troubleshooting](#troubleshooting)

---

## Overview

DeepDrishti provides an end-to-end solution for real-time monitoring of underwater environments. It is designed for naval and marine security use cases, providing the following capabilities:

| Capability | Description |
|---|---|
| **Underwater Image Enhancement** | FUNIE-GAN (Fast Underwater Image Enhancement using Generative Adversarial Networks) improves visibility in turbid, blue-shifted, or low-light underwater imagery |
| **Threat Detection** | Custom-trained YOLOv8 model identifies humans, submarines, sea mines, fish, trash, and submerged structures |
| **Depth Estimation** | Intel MiDaS `MiDaS_small` generates a monocular depth map for every frame, enabling approximate range estimation (up to ~15 m) for each detected object |
| **Real-time Feeds** | Multiple live camera feeds (YouTube embeds, RTSP/MP4 streams) are monitored at configurable intervals |
| **Role-Based Access** | Five operational roles (Captain → Analyst) with granular route-level enforcement |
| **Alerting & Audit** | Automatic alert generation, system log entries, and command-dispatch audit trail persisted in MongoDB |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      BROWSER / CLIENT                        │
│           React 19 + Vite  (http://localhost:5173)           │
└───────────────────────────┬─────────────────────────────────┘
                            │  /api/* → proxy
┌───────────────────────────▼─────────────────────────────────┐
│               NODE.JS / EXPRESS BACKEND                      │
│         AquaScope API  (http://localhost:5003)               │
│  Auth · Feeds · Alerts · Detections · Reports · Settings    │
└───────────┬────────────────────────┬────────────────────────┘
            │  Mongoose ODM          │  HTTP (axios / form-data)
┌───────────▼──────────┐  ┌─────────▼───────────────────────┐
│  MongoDB (Atlas/local)│  │   PYTHON / FLASK AI SERVICE     │
│  Collections:         │  │   DeepDrishti Pipeline          │
│  users, feeds,        │  │   (http://localhost:5001)        │
│  detections, alerts,  │  │                                  │
│  systemlogs,          │  │  1. FUNIE-GAN  Enhancement       │
│  commanddispatches,   │  │  2. YOLOv8    Detection          │
│  stationsettings      │  │  3. MiDaS     Depth Estimation  │
└──────────────────────┘  └──────────────────────────────────┘
```

---

## Technology Stack

### AI / Model Service
| Component | Technology |
|---|---|
| Web Framework | Flask 2.3+ with flask-cors |
| Image Enhancement | FUNIE-GAN (`GeneratorSmall` — U-Net style GAN, trained as `funie_final_improved.pth`) |
| Object Detection | YOLOv8 via `ultralytics` (`best.pt` — custom marine-threat classes) |
| Depth Estimation | Intel MiDaS `MiDaS_small` (loaded via `torch.hub`) |
| Image Processing | OpenCV, Pillow, scikit-image |
| Deep Learning Runtime | PyTorch 2.5+, torchvision, timm |
| Quality Metrics | PSNR, SSIM, UIQM |
| Stream Support | yt-dlp (YouTube), OpenCV VideoCapture (MP4/RTSP/M3U8) |

### Backend
| Component | Technology |
|---|---|
| Runtime | Node.js LTS |
| Framework | Express 4.18 |
| Database | MongoDB via Mongoose 8 |
| Authentication | JWT (`jsonwebtoken`) + bcryptjs password hashing |
| File Uploads | Multer |
| HTTP Hardening | Helmet, CORS, Morgan |
| Dev Server | Nodemon |

### Frontend
| Component | Technology |
|---|---|
| Framework | React 19 (Vite 7) |
| Routing | React Router DOM v7 |
| Styling | Tailwind CSS 3 + PostCSS |
| HTTP Client | Axios |
| PDF Export | jsPDF + jspdf-autotable |
| Build Tool | Vite with proxy to backend |

---

## Repository Structure

```
DeepDrishti/
├── README.md                    ← This file
│
├── model/                       ← Python AI pipeline (Flask)
│   ├── app.py                   ← Flask server entry point
│   ├── model.py                 ← UNet architecture definition
│   ├── model.pth                ← UNet weights (~83 MB)
│   ├── requirements.txt         ← Python dependencies
│   ├── models/
│   │   ├── funie_gan.py         ← FUNIE-GAN GeneratorSmall architecture
│   │   ├── funie_final_improved.pth  ← FUNIE-GAN weights (~5.5 MB)
│   │   ├── best.pt              ← YOLOv8 detection weights (~50 MB)
│   │   └── last.pt              ← YOLOv8 last-epoch checkpoint
│   ├── uploads/                 ← Uploaded images (runtime)
│   └── outputs/                 ← Enhanced output images (runtime)
│
├── backend/                     ← Node.js REST API
│   ├── package.json
│   └── server/
│       ├── server.js            ← Express app entry point
│       ├── .env.example         ← Environment variable template
│       ├── routes/              ← Express route files (14 files)
│       │   ├── authRoutes.js
│       │   ├── feedRoutes.js
│       │   ├── alertRoutes.js
│       │   ├── detectionRoutes.js
│       │   ├── dashboardRoutes.js
│       │   ├── reportRoutes.js
│       │   ├── systemLogRoutes.js
│       │   ├── settingsRoutes.js
│       │   ├── surveillanceRoutes.js
│       │   ├── aiEnhancementRoutes.js
│       │   ├── enhancementRoutes.js
│       │   ├── userRoutes.js
│       │   ├── testRoutes.js
│       │   └── testMarineRoutes.js
│       ├── controllers/         ← Business logic
│       ├── middleware/
│       │   ├── auth.js          ← JWT token extraction
│       │   ├── authMiddleware.js← Full JWT protect middleware
│       │   ├── authorize.js     ← RBAC role enforcement
│       │   └── errorHandler.js  ← Global error handler
│       ├── models/              ← Mongoose schemas
│       │   ├── User.js
│       │   ├── Feed.js
│       │   ├── Alert.js
│       │   ├── Detection.js
│       │   ├── Enhancement.js
│       │   ├── SystemLog.js
│       │   ├── CommandDispatch.js
│       │   ├── StationSettings.js
│       │   └── AccessRequest.js
│       ├── services/
│       │   └── detectionService.js  ← Automated polling detection loop
│       ├── seed/
│       │   ├── defaultUsers.js  ← Seeds 5 default users
│       │   └── marineFeeds.js   ← Seeds 4 sample camera feeds
│       └── utils/
│
└── frontend/                    ← React + Vite web dashboard
    ├── package.json
    ├── vite.config.js           ← Dev proxy: /api → localhost:5003
    ├── tailwind.config.js
    ├── index.html
    ├── public/
    │   └── videos/              ← Local demo video files
    └── src/
        ├── App.jsx              ← Route definitions
        ├── main.jsx
        ├── index.css
        ├── pages/               ← Top-level page components
        │   ├── Login.jsx
        │   ├── Dashboard.jsx
        │   ├── Surveillance.jsx
        │   ├── EnhancementLab.jsx
        │   ├── DetectionRecords.jsx
        │   ├── Reports.jsx
        │   ├── ReportsDetail.jsx
        │   ├── SystemLogs.jsx
        │   ├── Settings.jsx
        │   └── ConnectionTest.jsx
        ├── components/          ← Shared UI components (15 files)
        │   ├── Navbar.jsx
        │   ├── Sidebar.jsx
        │   ├── ProtectedRoute.jsx
        │   ├── FeedVideoPlayer.jsx
        │   ├── DetectionPreview.jsx
        │   ├── AnalyticsChart.jsx
        │   ├── ImagePanel.jsx
        │   └── ...
        ├── services/            ← Axios API client wrappers
        └── utils/               ← Helper utilities
```

---

## AI / ML Pipeline (`model/`)

The Flask service exposes a **three-stage pipeline** at `POST /pipeline` (alias `POST /enhance`):

### Stage 1 — FUNIE-GAN Image Enhancement

- **Architecture**: `GeneratorSmall` — a compact U-Net-style encoder-decoder GAN
  - Encoder: 3 downsampling layers (64 → 128 → 256 channels, Conv2d 4×4 stride-2 + BatchNorm + LeakyReLU)
  - Decoder: 2 upsampling layers with skip connections + final ConvTranspose2d + tanh
- **Input**: RGB image resized to 256×256, normalized to [-1, 1]
- **Output**: Enhanced image resized back to original resolution
- **Quality Metrics Computed**: PSNR, SSIM, UIQM

### Stage 2 — YOLOv8 Object Detection

- **Model**: Custom `best.pt` trained on marine-domain imagery
- **Confidence threshold**: 0.25 (at inference); a secondary station-level threshold (default 0.85) is enforced by the backend for alert generation
- **Detected classes** and their threat levels:

| Class | Display Name | Threat Level |
|---|---|---|
| `human` | human | DANGER |
| `mine` / `unexploded_ordnance` / `torpedo` | SEA MINE | DANGER |
| `submarine` / `sub` / `uuv` | SUBMERSIBLE | DANGER |
| `structure` | structure | NEUTRAL |
| `trash` | trash | NEUTRAL |
| `fish` | fish | SAFE |

### Stage 3 — MiDaS Depth Estimation

- **Model**: `MiDaS_small` (loaded via `torch.hub` from `intel-isl/MiDaS`)
- **Output**: Normalized depth map [0, 1]
- **Distance Formula**: `distance_m = (1 - normalised_depth) × 15`  
  → Assumes a maximum underwater visibility range of **15 metres**

### Input Sources Supported

| Source Type | How to Provide |
|---|---|
| Uploaded image file | `multipart/form-data` with field `image` or `file` |
| Image URL | JSON body `{ "url": "https://..." }` |
| Video URL / stream | JSON body `{ "url": "https://....mp4" }` |
| YouTube video/live | JSON body `{ "url": "https://youtube.com/..." }` (yt-dlp extracts stream) |
| Local video path | JSON body `{ "videoPath": "/absolute/path.mp4" }` |

### Flask API Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/` | Service info and available endpoints |
| `GET` | `/health` | Model status (funie, yolo, midas) and device (CPU/CUDA) |
| `POST` | `/pipeline` | Run full 3-stage AI pipeline |
| `POST` | `/enhance` | Alias for `/pipeline` |
| `GET` | `/outputs/<filename>` | Serve a previously generated enhanced image |

### Pipeline Response Shape

```json
{
  "success": true,
  "data": {
    "enhanced_image_path": "/outputs/pipeline_1713300000.jpg",
    "output_filename": "pipeline_1713300000.jpg",
    "image_width": 1920,
    "image_height": 1080,
    "metrics": {
      "psnr": 28.45,
      "ssim": 0.812,
      "uiqm": 0.563
    },
    "detections": [
      {
        "class": "SEA MINE",
        "raw_class": "mine",
        "confidence": 0.87,
        "bbox": [120, 200, 350, 410],
        "status": "DANGER",
        "distance_m": 4.2,
        "models_used": ["FUNIE-GAN", "YOLOv8", "MiDaS"]
      }
    ],
    "processing_time_ms": 1243.5,
    "model_device": "cuda"
  }
}
```

### Running the Model Service

```bash
cd model
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

pip install -r requirements.txt
python app.py
# Service starts on http://0.0.0.0:5001
```

---

## Backend API (`backend/`)

Built with **Express 4** and **Mongoose 8**, the Node.js backend provides the full REST API consumed by the React frontend.

### Key Features

- **Resilient MongoDB connection** with exponential back-off retry (up to 30 s intervals)
- **JWT-based authentication** using `jsonwebtoken` + `bcryptjs` (salt rounds: 12)
- **RBAC enforcement** at the route level via `middleware/authorize.js`
- **Automated detection polling** via `services/detectionService.js` — starts only after MongoDB is confirmed connected
- **Helmet** HTTP security headers
- **Multer** for multipart file uploads (max 10 MB)
- **Morgan** request logging

### Running the Backend

```bash
cd backend
npm install
# Copy and configure environment
copy server\.env.example server\.env
# Edit server\.env with your MongoDB URI and JWT secret
npm run dev
# API server runs on http://localhost:5003
```

---

## Frontend Dashboard (`frontend/`)

A **React 19 + Vite 7** single-page application styled with **Tailwind CSS 3**.

### Pages & Routes

| Route | Component | Access |
|---|---|---|
| `/` or `/login` | `Login.jsx` | Public |
| `/dashboard` | `Dashboard.jsx` | All authenticated roles |
| `/surveillance` | `Surveillance.jsx` | `surveillance_head` and above |
| `/enhancement-lab` | `EnhancementLab.jsx` | `engineer` and above |
| `/detection-records` | `DetectionRecords.jsx` | All authenticated roles |
| `/reports` | `Reports.jsx` | `surveillance_head` and above |
| `/reports/detail` | `ReportsDetail.jsx` | `surveillance_head` and above |
| `/system-logs` | `SystemLogs.jsx` | `engineer` and above |
| `/settings` | `Settings.jsx` | All authenticated roles |
| `/test-connection` | `ConnectionTest.jsx` | Public (dev utility) |

### Key Pages Explained

#### Dashboard
Displays a real-time summary: active alerts count, total detections, feed status, system health. Pulls from `/api/v1/dashboard/summary`.

#### Surveillance
Live multi-feed monitoring interface. Operators can:
- Watch multiple camera feeds simultaneously
- Pause/resume individual feeds
- Mark regions of interest (ROI) on a feed
- Trigger on-demand AI analysis for a specific feed
- Raise manual alerts

#### Enhancement Lab
Upload an image or point to a live feed URL; the UI sends the payload to the Node backend which proxies it to the Flask AI service. Results show the enhanced image side-by-side with detection boxes and quality metrics (PSNR, SSIM, UIQM).

#### Reports
Analytics view driven by `/api/v1/reports/analytics`. Includes a **Send to Command** button that calls `POST /api/v1/reports/dispatch`, which persists:
1. A `SystemLog` document (module: `COMM-RELAY`)
2. A `CommandDispatch` document linked by `systemLogId`

#### System Logs
Tabular view of all system-level events (severity: INFO / WARN / ERROR / CRITICAL), filterable and paginated.

#### Settings
Configure station-wide AI parameters:
- Enhancement strength, dehaze factor, color correction, low-light boost
- Water turbidity preset
- Confidence threshold for alert generation
- Debris detection, diver recognition, subsurface detection toggles
- Exclusion list (e.g., "Fish Shoals", "Sea Kelp")
- Alert cooldown, FPS limit, archiving options

### Running the Frontend

```bash
cd frontend
npm install
npm run dev
# UI available at http://localhost:5173
# /api/* requests are proxied to http://127.0.0.1:5003
```

---

## Database Schema

### `users` Collection

| Field | Type | Notes |
|---|---|---|
| `name` | String | Full display name |
| `email` | String | Unique, lowercase |
| `password` | String | bcrypt-hashed (not returned in queries) |
| `role` | String | Enum: `captain`, `vice_captain`, `surveillance_head`, `engineer`, `analyst` |
| `isActive` | Boolean | Soft disable |
| `lastLogin` / `lastActivity` | Date | Activity tracking |
| `preferences` | Object | `theme`, `notifications`, `language` |

### `feeds` Collection

| Field | Type | Notes |
|---|---|---|
| `title` | String | Camera name |
| `url` | String | YouTube embed or stream URL |
| `status` | String | `active`, `inactive`, `maintenance` |
| `location` | String | Geographical description |
| `sector` | String | Operational zone (Alpha, Beta, …) |
| `detectionEnabled` | Boolean | Toggle per-feed AI polling |
| `detectionInterval` | Number | Polling interval in ms (5 000 – 60 000) |
| `surveillance` | Object | `markedRegion`, `streamPaused`, timestamps |

### `detections` Collection

| Field | Type | Notes |
|---|---|---|
| `objectDetected` | String | Class name |
| `confidence` | Number | 0 – 100 |
| `threatStatus` | String | `DANGER`, `NEUTRAL`, `SAFE`, `CLEAR` |
| `distance_m` | Number | MiDaS-estimated distance |
| `modelsUsed` | [String] | `["FUNIE-GAN", "YOLOv8", "MiDaS"]` |
| `boundingBox` | Mixed | `{x1, y1, x2, y2}` |
| `feedId` | ObjectId | Reference to feed |
| `snapshotPath` / `enhancedImage` | String | Image file paths |

### `alerts` Collection

| Field | Type | Notes |
|---|---|---|
| `type` | String | `intrusion`, `anomaly`, `object`, `threat` |
| `severity` | String | Auto-computed from confidence: low/medium/high/critical |
| `status` | String | `active`, `investigating`, `resolved` |
| `detectionData` | Object | Snapshot of detection (bbox, distance, models, …) |
| `resolvedBy` | ObjectId | User who resolved |
| `assignedTo` | ObjectId | Assigned personnel |

### `systemlogs` Collection

Audit log for all significant events (severity: `INFO`, `WARN`, `ERROR`, `CRITICAL`). Module field tags the source subsystem (e.g., `COMM-RELAY`, `AI-PIPELINE`).

### `commanddispatches` Collection

Created on every **Send to Command** action. Links to a `SystemLog` document via `systemLogId` for full audit traceability.

### `stationsettings` Collection (Singleton)

A single document holding all configurable AI/detection parameters for the station (see Settings page above).

---

## RBAC — Roles & Permissions

| Role | Level | Capabilities |
|---|---|---|
| `captain` | Highest | Full read/write access to all resources |
| `vice_captain` | 2 | Same as captain, with operational scope |
| `surveillance_head` | 3 | Surveillance, feeds, alerts, reports, detections |
| `engineer` | 4 | Enhancement lab, system logs, technical settings |
| `analyst` | Lowest | **Read-only** — all mutating HTTP methods (POST/PUT/PATCH/DELETE) are blocked |

Route-level enforcement is applied by the `authorize(...roles)` middleware in `backend/server/middleware/authorize.js`. The frontend enforces role-based routing via the `<ProtectedRoute requiredRole="...">` wrapper component.

---

## API Reference

All endpoints are prefixed with `/api/v1`. Protected routes require the header:
```
Authorization: Bearer <jwt_token>
```

### Authentication

| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/login` | Authenticate and receive JWT |
| `GET` | `/auth/me` | Return current user profile |
| `POST` | `/auth/logout` | Invalidate session |

### Feeds

| Method | Path | Description |
|---|---|---|
| `GET` | `/feeds` | List all camera feeds |
| `POST` | `/feeds` | Create a new feed |
| `GET` | `/feeds/:id` | Get single feed |
| `PUT` | `/feeds/:id` | Update feed config |
| `DELETE` | `/feeds/:id` | Remove feed |
| `POST` | `/feeds/:id/surveillance/ai-analysis` | Trigger on-demand AI analysis |
| `POST` | `/feeds/:id/surveillance/capture` | Capture frame snapshot |
| `POST` | `/feeds/:id/surveillance/mark-area` | Save ROI coordinates |
| `POST` | `/feeds/:id/surveillance/pause` | Pause/resume stream |

### Alerts

| Method | Path | Description |
|---|---|---|
| `GET` | `/alerts` | List all alerts (paginated) |
| `GET` | `/alerts/active` | Active alerts only |
| `POST` | `/alerts` | Create manual alert |
| `PATCH` | `/alerts/:id/resolve` | Resolve an alert |

### Detections

| Method | Path | Description |
|---|---|---|
| `GET` | `/detections` | List all detection records |
| `GET` | `/detections/:id` | Get single detection |
| `PATCH` | `/detections/:id/confirm` | Mark as confirmed |
| `PATCH` | `/detections/:id/false-positive` | Mark as false positive |

### Dashboard

| Method | Path | Description |
|---|---|---|
| `GET` | `/dashboard/summary` | Aggregated counts (alerts, detections, feeds) |
| `GET` | `/dashboard/surveillance/:feedId` | Per-feed surveillance summary |
| `GET` | `/dashboard/reports-analytics` | Analytics data mirror |
| `POST` | `/dashboard/reports-dispatch` | Dispatch to command (mirror route) |

### Reports

| Method | Path | Description |
|---|---|---|
| `GET` | `/reports/analytics` | Full analytics dataset |
| `POST` | `/reports/dispatch` | Send report to command (creates SystemLog + CommandDispatch) |

### System Logs

| Method | Path | Description |
|---|---|---|
| `GET` | `/system-logs` | Paginated system log entries |

### Settings

| Method | Path | Description |
|---|---|---|
| `GET` | `/settings` | Get current station settings |
| `PUT` | `/settings` | Update station settings |

### AI Enhancement

| Method | Path | Description |
|---|---|---|
| `POST` | `/ai-enhance` | Proxy multipart image to Flask pipeline and return AI results |
| `POST` | `/enhance` | Basic enhancement endpoint |

### Health

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | API health check |

---

## Configuration & Environment Variables

### Backend (`backend/server/.env`)

Copy `backend/server/.env.example` to `backend/server/.env` and fill in:

```env
# Required
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/aquascope
JWT_SECRET=your-strong-random-secret-min-32-chars

# Optional (defaults shown)
PORT=5003
NODE_ENV=development
API_VERSION=1.0.0
FRONTEND_URL=http://localhost:5173

# File Uploads
MAX_FILE_SIZE=10485760        # 10 MB
UPLOAD_PATH=uploads

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000   # 15 min
RATE_LIMIT_MAX_REQUESTS=100

# Security
BCRYPT_SALT_ROUNDS=12
JWT_EXPIRE_TIME=1d
JWT_EXPIRE_TIME_REMEMBER=7d
```

### Frontend (`frontend/.env`)

```env
VITE_API_URL=/api/v1
# Optional: override proxy target
VITE_DEV_API_ORIGIN=http://127.0.0.1:5003
```

> **Note:** After changing any `.env` file, restart the respective dev server.

---

## Getting Started

### Prerequisites

- **Node.js** LTS (≥ 18)
- **Python** 3.10 – 3.12
- **MongoDB** — Atlas cluster (recommended) or local `mongod`
- **Git**

### 1. Clone the repository

```bash
git clone https://github.com/<your-org>/DeepDrishti.git
cd DeepDrishti
```

### 2. Start the Python AI Service

```bash
cd model
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS/Linux

pip install -r requirements.txt
python app.py
# Listening on http://0.0.0.0:5001
```

### 3. Start the Backend API

```bash
cd backend
npm install
# Ensure backend/server/.env is configured (see above)
npm run dev
# API running on http://localhost:5003
```

### 4. Start the Frontend

```bash
cd frontend
npm install
npm run dev
# UI at http://localhost:5173
```

Open **http://localhost:5173** and log in with one of the seeded accounts below.

---

## Seeding the Database

### Seed default users (5 accounts)

```bash
cd backend
npm run seed
```

### Seed sample camera feeds (4 feeds)

```bash
cd backend
node server/seed/marineFeeds.js
```

---

## Default Login Credentials

> ⚠️ Change all passwords before any production deployment.

| Role | Email | Password |
|---|---|---|
| `captain` | captain@aquascope.com | `Captain@123` |
| `vice_captain` | vice@aquascope.com | `Vice@123` |
| `surveillance_head` | surveillance@aquascope.com | `Surv@123` |
| `engineer` | engineer@aquascope.com | `Eng@123` |
| `analyst` | analyst@aquascope.com | `Analyst@123` |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| **404 on `/api/v1/...`** | Confirm the Node server is running from `backend/` and all routes are mounted in `server.js`. Restart after pulling changes. |
| **CORS / wrong port errors** | Align `PORT` in `backend/server/.env`, `VITE_API_URL` in `frontend/.env`, and the `server.proxy` target in `frontend/vite.config.js`. |
| **Empty System Logs after dispatch** | Confirm `MONGODB_URI` points to the correct database. Check the Network tab for `500` on the dispatch call — a write failure returns HTTP 500. |
| **Flask models not loading** | Check that `model/models/funie_final_improved.pth` and `model/models/best.pt` exist. Run `GET http://localhost:5001/health` to see per-model status. |
| **CUDA / GPU not available** | Flask service falls back to CPU automatically. Ensure CUDA drivers and the CUDA-enabled PyTorch wheel are installed if GPU acceleration is desired. |
| **MongoDB Atlas IP whitelist** | Add your current public IP in Atlas **Network Access**. The backend will log retry attempts if the initial connection fails. |
| **yt-dlp YouTube errors** | Update yt-dlp: `pip install -U yt-dlp`. YouTube frequently changes its API. |

---

## License

See project coursework or team policy; update this section as required.

---

*Built by the DeepDrishti Team — Marine AI Surveillance Stack.*
