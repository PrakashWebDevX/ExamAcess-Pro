# Online Examination Platform

> Secure, scalable, mobile-friendly online exams with advanced proctoring and a real-time admin dashboard.

## Overview

This repository contains the code and infrastructure for a comprehensive online examination platform designed for educational institutions. Key goals:

---
This Demo Website: ( https://examaccesspro.netlify.app/ )

---

* Provide secure and scalable exam delivery.
* Offer real-time administrative monitoring and analytics.
* Integrate advanced AI-powered proctoring for integrity.
* Maintain compliance and audit-ready reporting.

---

## Table of Contents

1. [Features](#features)
2. [Architecture](#architecture)
3. [Tech Stack](#tech-stack)
4. [Getting Started](#getting-started)

   * [Prerequisites](#prerequisites)
   * [Local Development](#local-development)
   * [Docker / Docker Compose](#docker--docker-compose)
5. [Scripts & Example Files](#scripts--example-files)

   * `docker-compose.yml`
   * `setup.sh` (bootstrap)
   * `migrate.sh` (DB migrations)
   * `package.json` scripts
   * GitHub Actions workflow: `ci.yml`
6. [Admin Dashboard & Analytics](#admin-dashboard--analytics)
7. [Proctoring Integrations](#proctoring-integrations)
8. [Security & Compliance](#security--compliance)
9. [API Endpoints (example)](#api-endpoints-example)
10. [Deployment Recommendations](#deployment-recommendations)
11. [Testing & Monitoring](#testing--monitoring)
12. [FAQ & Troubleshooting](#faq--troubleshooting)
13. [Critique of this README & approach](#critique-of-this-readme--approach)

---

## Features

* Live system health monitoring (uptime, CPU/RAM, response times)
* Real-time user activity analytics (session heatmaps, peak times)
* Role-based access control (students, proctors, instructors, admins)
* Multi-factor authentication (OTP, biometric metadata tracking)
* AI proctoring: eye tracking (WebGazer.js / jsPsych), continuous face detection, motion analysis, audio analysis and diarization
* Compliance reporting & audit logs
* Mobile-friendly proctoring (progressive web app + native wrappers)

---

## Architecture

**High-level components**:

* **Frontend**: React + Tailwind (or similar) PWA for exam UI and admin dashboard.
* **Proctoring Worker**: lightweight client-side JS modules to gather webcam/audio sensors and stream events (not raw video for privacy-options) to backend.
* **Backend API**: Node.js/Express or Python/FastAPI for authentication, exam orchestration, event ingestion.
* **Realtime Layer**: WebSocket (Socket.IO) or WebTransport for live heartbeats and activity stream.
* **Event Store / Analytics**: Kafka or Redis Streams → processing (Flink/Beam or Node workers) → Timeseries DB (InfluxDB / ClickHouse) + OLAP (ClickHouse / Postgres+materialized views).
* **Object Storage**: S3 (or compatible) for recorded artifacts (if storing) with encryption at rest.
* **DB**: Postgres primary for relational data + Redis for session cache and rate-limiting.
* **AI Services**: hosted model endpoints (on-prem or cloud) for face recognition, voice biometrics, diarization, and motion classification.

---

## Tech Stack (example)

* Frontend: React, Vite, Tailwind, WebGazer.js, jsPsych
* Backend: Node.js (TypeScript) + Express/Fastify or Python FastAPI
* Database: PostgreSQL, Redis
* Streams & Analytics: Kafka / Redis Streams, ClickHouse / InfluxDB
* Storage: MinIO / AWS S3
* Containerization: Docker, Kubernetes (EKS/GKE/AKS)
* CI/CD: GitHub Actions
* Monitoring: Prometheus + Grafana, Sentry, BoldBI for dashboards

---

## Getting Started

### Prerequisites

* Node.js 18+ (or Python 3.10+ if using FastAPI stack)
* Docker & Docker Compose (for local all-in-one)
* PostgreSQL (or use Docker)
* Git

### Local Development (quick)

1. Clone repository:

```bash
git clone git@github.com:yourorg/online-exam-platform.git
cd online-exam-platform
```

2. Copy env and configure:

```bash
cp .env.example .env
# Edit .env to configure DB, secrets, S3 endpoints
```

3. Start services:

```bash
# if using docker-compose
docker-compose up --build

# or run frontend and backend separately
cd frontend
npm install
npm run dev

cd ../backend
pnpm install
pnpm dev
```

### Docker / Docker Compose

A sample `docker-compose.yml` is included in `/scripts` to bring up: Postgres, Redis, MinIO, and the backend+frontend in development mode.

---

## Scripts & Example Files

**Included example scripts** (place in `/scripts`):

* `setup.sh` — bootstrap local env, create DB user, run migrations, seed demo data.
* `migrate.sh` — wrapper around `knex`/`alembic`/`prisma migrate`.
* `docker-compose.yml` — local stack for dev.
* `ci.yml` — GitHub Actions pipeline skeleton.

**Example `package.json` scripts (frontend)**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.tsx,.js,.jsx",
    "test": "vitest"
  }
}
```

**Example GitHub Actions (simplified)**

```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: pnpm install --frozen-lockfile
      - run: pnpm test
      - run: pnpm build --filter=frontend
```

---

## Admin Dashboard & Analytics

Dashboard components:

* System Health Overview: uptime, latency percentiles, error rates
* Performance Timeline: 24-hour rolling metrics, request traces
* Activity Map: concurrent users, active exams, session durations
* Security Panel: MFA usage metrics, failed authentication heatmap, suspicious behavior alerts
* Audit & Report Builder: export CSV/PDF for given window

**Visual tools**: integrate BoldBI or jsCharting widgets + custom React visualization components that query ClickHouse/Timeseries store for fast returns.

---

## Proctoring Integrations

**Client-side modules** (run inside the exam iframe or PWA):

* **WebGazer.js**: gaze estimation on device using webcam. Use for suspicious gaze detection; do NOT rely on it alone.
* **jsPsych Eye Tracking**: for experiment-style controlled stimuli and calibration flows.
* **Motion Detection**: small JS worker using frame-differencing to detect large motions.
* **Audio**: collect short audio segments (or features) for diarization/voice biometrics using Web Audio API; avoid storing raw audio unless required—store features or encrypted chunks.

**Privacy note**: Always surface consent and data retention policies. Offer low-privacy mode (no raw video stored; only behavioral features and flags).

---

## Security & Compliance

* Enforce RBAC: central `roles` table plus fine-grained permissions mapping.
* Harden transports: TLS everywhere, HSTS, secure cookies.
* Encryption: at-rest (S3 + DB) and in-transit (TLS). Rotate keys regularly.
* Audit logs: append-only event store with tamper-evident hashing for critical actions.
* GDPR/FERPA considerations: data minimization, retention windows, right-to-delete workflows.

---

## API Endpoints (example)

```
POST /api/v1/auth/login
POST /api/v1/auth/mfa/verify
GET  /api/v1/admin/system/health
GET  /api/v1/admin/analytics/active-sessions
POST /api/v1/exams/:examId/start
POST /api/v1/exams/:examId/events   # ingest proctoring events
GET  /api/v1/reports/export
```

Use JWT access tokens + rotating refresh tokens and scopes per role.

---

## Deployment Recommendations

* Use Kubernetes for scaling proctoring workers and API pods.
* Separate proctoring heavy processing into a worker cluster that consumes event streams and emits flags.
* Use auto-scaling on CPU + custom metrics (concurrent exams) to avoid overload during peak exam windows.
* Implement blue/green or canary releases for safe updates.

---

## Testing & Monitoring

* Unit & integration tests for core flows (auth, exam life-cycle).
* E2E tests using Playwright for UI flows and proctoring fallbacks (simulate camera off, mic muted).
* Load testing for peak exam sizes (k6 / Locust).
* Observability: Prometheus metrics + Grafana dashboards; Sentry for errors. Correlate proctoring flags with session traces.

---

## FAQ & Troubleshooting

**Q: What if a student’s camera fails?**
A: Provide immediate fallback: allow proctor override, pause exam with integrity log, and record device failure event with screenshot attempt counts.

**Q: Can proctoring be disabled?**
A: Yes — institution-level policy toggles with an audit trail. Disabling still records general activity logs.

---

## LICENSE

MIT

---
# Online Examination Platform

> Secure, scalable, mobile-friendly online exams with advanced proctoring and a real-time admin dashboard.

## 📁 Project Structure

```
frontend/
│
├── public/                # Static assets (index.html, favicon, images, etc.)
│
├── src/
│   ├── index.js           # React app entry point
│   ├── App.js             # Main app component
│   ├── components/        # Reusable UI components (Navbar, Button, Modal, etc.)
│   ├── pages/             # Page-level views (Dashboard, Exam, Login, Reports)
│   ├── hooks/             # Custom React hooks (useAuth, useFetch, etc.)
│   ├── context/           # Global context providers (AuthContext, ThemeContext)
│   ├── services/          # API service calls (axios instance, endpoints)
│   ├── utils/             # Utility functions (validators, formatters)
│   └── styles/            # Global styles, Tailwind config, themes
│
├── package.json           # Frontend dependencies and scripts
├── vite.config.js         # Vite configuration (or CRA alternative)
└── README.md              # Frontend guide

backend/
│
├── app/
│   ├── Controllers/       # Route handlers
│   ├── Models/            # Database models (User, Exam, Submission)
│   ├── Middleware/        # Auth, Logging, Error handling
│   ├── Routes/            # Express/FastAPI routes
│   ├── Services/          # Business logic (Proctoring, Reports, Analytics)
│   ├── config/            # DB, Environment, JWT, Logging
│   ├── utils/             # Helper scripts (token helpers, hashing)
│   └── main.js            # App entry point
│
├── tests/                 # Unit & integration tests
├── package.json / pyproject.toml
├── Dockerfile
└── README.md

scripts/
│
├── docker-compose.yml     # Orchestration for backend + frontend + DB + Redis
├── setup.sh               # Bootstrapping script for local development
├── migrate.sh             # Database migration utility
└── ci.yml                 # GitHub Actions workflow (CI/CD)
```

---

## 🧠 Features

* Real-time analytics dashboard for admins
* Role-based access control (RBAC)
* AI-powered proctoring (WebGazer.js, jsPsych Eye Tracking)
* Multi-factor authentication (OTP, biometric)
* Secure RESTful API
* Audit logging and compliance reporting
* Mobile-friendly UI (React PWA)

---

## ⚙️ Setup & Installation

### Prerequisites

* Node.js 18+
* Docker & Docker Compose
* PostgreSQL & Redis (for local or via Docker)

### Steps

```bash
git clone https://github.com/yourusername/online-exam-platform.git
cd online-exam-platform
```

1. **Environment setup**

```bash
cp .env.example .env
# Fill in DB credentials, JWT_SECRET, and API_BASE_URL
```

2. **Run setup script**

```bash
chmod +x scripts/setup.sh
./scripts/setup.sh
```

3. **Start with Docker Compose**

```bash
docker-compose up --build
```

4. **Access locally:**

* Frontend: `http://localhost:5173`
* Backend API: `http://localhost:5000/api`

---

## 🔍 Example `scripts/setup.sh`

```bash
#!/bin/bash
echo "Setting up Online Exam Platform..."

# Install dependencies
cd backend && npm install && cd ../frontend && npm install

# Run DB migrations
cd ../backend && npm run migrate

# Start Docker containers
docker-compose up --build -d

echo "Setup complete! Visit http://localhost:5173 to begin."
```

---

## 🧪 Testing

```bash
# Frontend tests
cd frontend && npm run test

# Backend tests
cd backend && npm run test
```

Continuous Integration (CI) is handled by GitHub Actions (`.github/workflows/ci.yml`) to lint, test, and build both services automatically on push.

---

## 🚀 Deployment

* Netlify
* stack: AWS ECS / GCP Cloud Run / Azure App
