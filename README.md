```css
████████╗██╗  ██╗██████╗ ███████╗ █████╗ ████████╗
╚══██╔══╝██║  ██║██╔══██╗██╔════╝██╔══██╗╚══██╔══╝
   ██║   ███████║██████╔╝█████╗  ███████║   ██║
   ██║   ██╔══██║██╔══██╗██╔══╝  ██╔══██║   ██║
   ██║   ██║  ██║██║  ██║███████╗██║  ██║   ██║
   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝   ╚═╝
```

# AI-Powered Threat Detection Engine

AI-powered threat detection engine that analyzes nginx access logs using a 3-model ML ensemble to classify HTTP traffic as benign or malicious in real time.

## Features

- **3-Model ML Ensemble:** Autoencoder, Random Forest, and Isolation Forest exported to ONNX for fast CPU inference.
- **4-Stage Async Pipeline:** Parse raw logs, extract 35-dimensional feature vectors, score with rules + ML, dispatch alerts.
- **Rule Engine:** ModSecurity CRS-inspired patterns (SQLi, XSS, path traversal, command injection, Log4Shell, SSRF).
- **Real-Time Alerts:** WebSocket feed with live severity scoring and threat classification.
- **Auto-Training:** Trains on first deploy with synthetic data; supports retraining from the dashboard using actual stored events.
- **Docker Sidecar:** Deploys alongside any nginx-based infrastructure with zero code changes.

## Quick Start

```bash
git clone https://github.com/Mxnojx4/AI-Threat-Detection.git
cd AI-Threat-Detection
cp .env.example .env
docker compose up -d
First startup takes ~2 minutes while models train. Visit http://localhost:46969 for the dashboard once healthy.

Note: This project uses just as a command runner. Type just to see all available commands.

Simulate Attacks
Use the built-in dev-log application to generate realistic nginx traffic for testing:

bash
Download
Copy code
docker compose -f dev-log/compose.yml up -d
python dev-log/simulate.py mixed -n 100
Available Attack Modes: normal, sqli, xss, traversal, cmdi, log4shell, ssrf, scanner, flood, mixed

Architecture
text
Download
Copy code
┌─────────────────────────────────────────────────────────────┐
│                        nginx logs                           │
│                     (shared volume)                         │
└──────────────────────────┬──────────────────────────────────┘
                           │
                    PollingObserver
                           │
                   ┌───────▼───────┐
                   │   Raw Queue   │
                   └───────┬───────┘
                           │
              ┌────────────▼────────────┐
              │     Stage 1: Parse      │
              │  nginx combined format  │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │   Stage 2: Features     │
              │  35-dim vector + GeoIP  │
              │  + windowed aggregates  │
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │   Stage 3: Detection    │
              │  Rules + ML Ensemble    │
              │  AE(40%) RF(40%) IF(20%)│
              └────────────┬────────────┘
                           │
              ┌────────────▼────────────┐
              │   Stage 4: Dispatch     │
              │  PostgreSQL + Redis     │
              │  pub/sub → WebSocket    │
              └─────────────────────────┘
Threat Scoring (0.0 - 1.0):

HIGH (0.7+): Stored, alerted via WebSocket, block recommendation.
MEDIUM (0.5-0.7): Stored, monitored.
LOW (<0.5): Logged for pattern analysis.
API Endpoints
Endpoint	Description
GET /health	Health check
GET /stats	Threat statistics and severity breakdown
GET /threats	Paginated threat events with filters
GET /models/status	Active models, detection mode, metrics
POST /models/retrain	Trigger retraining from stored events
POST /ingest/batch	Manual log line ingestion
WS /ws/alerts	Real-time threat alert stream
All endpoints (except health and WebSocket) require X-API-Key header.

Tech Stack
Backend: FastAPI (async), PostgreSQL 18, Redis 7.4, ONNX Runtime, PyTorch, scikit-learn, MLflow
Frontend: React 19, TypeScript, Vite, Sass, TanStack Query, Zustand
Infra: Docker Compose, multi-stage builds, auto-training entrypoint, shared volume log tailing
