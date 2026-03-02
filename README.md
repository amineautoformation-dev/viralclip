<div align="center">

# ✂️ ViralClip

**AI-Powered Video Clipping Engine — Turn any YouTube video into viral short-form clips**

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Next.js](https://img.shields.io/badge/Next.js-15-000000?style=for-the-badge&logo=next.js&logoColor=white)](https://nextjs.org/)
[![Celery](https://img.shields.io/badge/Celery-5.4+-37814A?style=for-the-badge&logo=celery&logoColor=white)](https://docs.celeryq.dev/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)](LICENSE)

[Features](#-features) · [Architecture](#-architecture) · [Quick Start](#-quick-start) · [API Reference](#-api-reference) · [Contributing](#-contributing)

</div>

---

## 🎯 What is ViralClip?

ViralClip is a **full-stack SaaS platform** that transforms long-form YouTube videos (podcasts, interviews, lectures, vlogs) into optimized short-form clips for TikTok, Instagram Reels, YouTube Shorts, and Facebook.

**Drop a YouTube URL → Get ready-to-post viral clips in minutes.**

The platform automates the entire workflow:

```
YouTube URL → Download → Transcribe → AI Analysis → Face Tracking → Subtitles → Export
```

### What makes ViralClip different?

| Feature | ViralClip | Opus Clip | Manual Editing |
|---------|:---------:|:---------:|:--------------:|
| Re-clips existing videos | ✅ | ✅ | ✅ |
| Smart face tracking | ✅ | ❌ | ❌ |
| Hormozi-style subtitles | ✅ | ❌ | Manual |
| Multi-engine AI fallback | ✅ | ❌ | ❌ |
| Multi-platform export (9:16, 4:5) | ✅ | ✅ | Manual |
| SEO-optimized descriptions | ✅ | ❌ | Manual |
| Self-hosted / Open source | ✅ | ❌ | N/A |
| Cost per video | ~$0.02 | $0.50+ | $15+ (time) |

---

## ✨ Features

### 🧠 Multi-Engine AI Analysis
Automatically selects the best available AI engine with intelligent fallback:
- **Gemini 2.0 Flash** — Primary (fast, high quality)
- **DeepSeek V3** — Secondary
- **Groq Llama 3.3** — Tertiary (ultra-fast inference)
- **Alibaba Qwen Plus** — Quaternary

### 👤 Smart Face Tracking
MediaPipe-powered face detection that keeps speakers perfectly centered in vertical crops, even in multi-person scenarios.

### 📝 Hormozi-Style Dynamic Subtitles
Word-by-word animated subtitles with semi-transparent background — the proven format that boosts retention by 40%.

### 🎬 AI Effects Director
An AI-generated artistic plan drives automatic zoom, transitions, hook flash effects, and audio normalization per platform.

### 📱 Multi-Platform Export
A single pipeline generates platform-optimized versions:
- **TikTok** — 9:16, vibrant color grading
- **Instagram Reels** — 9:16, warm tones
- **Facebook** — 4:5, balanced palette
- **YouTube Shorts** — 9:16, neutral

### 🔍 SEO-Ready Output
AI generates optimized titles, descriptions, hashtags, and tags for each clip and each platform.

### ⚡ Async Processing
Celery + Redis workers handle video processing in the background. Submit jobs via the API or dashboard, poll for status, and download results.

---

## 🏗 Architecture

ViralClip follows a **microservices architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                             │
│              Next.js 15 · Tailwind CSS · shadcn/ui          │
│                    http://localhost:3001                     │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API
┌────────────────────────▼────────────────────────────────────┐
│                      API GATEWAY                            │
│             FastAPI · Pydantic · CORS                       │
│                    http://localhost:8080                     │
└──────┬─────────────────────────────────────────┬────────────┘
       │ Celery Task                             │ SQL
┌──────▼──────────┐                   ┌──────────▼───────────┐
│   TASK QUEUE    │                   │     DATABASE         │
│  Redis 7 Alpine │                   │  PostgreSQL 15       │
│   :6379         │                   │   :5433              │
└──────┬──────────┘                   └──────────────────────┘
       │
┌──────▼──────────────────────────────────────────────────────┐
│                     CELERY WORKER                           │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │
│  │Download  │→ │Transcribe│→ │AI Analyze│→ │Face Track  │  │
│  │(yt-dlp)  │  │(Whisper) │  │(Multi-AI)│  │(MediaPipe) │  │
│  └──────────┘  └──────────┘  └──────────┘  └─────┬──────┘  │
│                                                   │         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────▼──────┐  │
│  │SEO Gen   │← │Effects   │← │Subtitles │← │Clip & Crop │  │
│  │(AI)      │  │(Director)│  │(ASS/SRT) │  │(FFmpeg)    │  │
│  └──────────┘  └──────────┘  └──────────┘  └────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

> 📐 For detailed architecture diagrams, see [ARCHITECTURE.md](ARCHITECTURE.md).

---

## 📁 Project Structure

```
viralclip/
├── api/
│   └── main.py              # FastAPI application (REST endpoints)
├── worker/
│   ├── __init__.py
│   └── tasks.py             # Celery async task definitions
├── core/
│   ├── celery_app.py         # Celery + Redis configuration
│   ├── config.py             # Pydantic settings (YAML + env)
│   └── logger.py             # Structured colored logging
├── engines/                  # AI engine adapters (strategy pattern)
│   ├── base.py               # Abstract base engine
│   ├── gemini.py             # Google Gemini 2.0 Flash
│   ├── deepseek.py           # DeepSeek V3
│   ├── groq.py               # Groq Llama 3.3
│   ├── alibaba.py            # Alibaba Qwen Plus
│   └── engine_manager.py     # Auto-selection & fallback logic
├── frontend/                 # Next.js 15 SaaS dashboard
│   └── src/
│       ├── app/
│       │   ├── page.tsx      # Landing page (hero, features, pricing)
│       │   ├── dashboard/
│       │   │   └── page.tsx  # Job submission & monitoring
│       │   ├── layout.tsx    # Root layout (SEO, fonts, dark mode)
│       │   └── globals.css   # Design system (emerald/purple palette)
│       ├── components/
│       │   ├── navbar.tsx    # Sticky nav with blur backdrop
│       │   └── ui/           # shadcn/ui components
│       └── lib/
│           └── api.ts        # Typed API client (fetch wrapper)
├── main.py                   # CLI entry point & pipeline orchestrator
├── downloader.py             # YouTube download (yt-dlp)
├── transcriber.py            # Audio transcription (Whisper / Groq)
├── analyzer.py               # Viral moment detection (multi-AI)
├── effects_director.py       # AI-driven effects planning
├── clipper.py                # Video clipping & cropping (FFmpeg)
├── face_tracker.py           # Face detection (MediaPipe)
├── subtitle_generator.py     # ASS/SRT subtitle generation
├── seo_generator.py          # SEO title/desc/hashtag generation
├── schemas.py                # Pydantic data models
├── config/
│   └── settings.yaml         # Pipeline configuration
├── docker-compose.yml        # 6-service orchestration
├── Dockerfile                # Python backend image
├── requirements.txt          # Python dependencies
├── .env.example              # Environment template
├── ARCHITECTURE.md           # Detailed system design
├── CONTRIBUTING.md           # Contribution guidelines
└── LICENSE                   # MIT License
```

---

## 🚀 Quick Start

### Prerequisites

- **Docker** & **Docker Compose** v2+
- At least **one AI API key**: Gemini, DeepSeek, Groq, or Alibaba

### 1. Clone & Configure

```bash
git clone https://github.com/amineautoformation-dev/viralclip.git
cd viralclip

# Copy environment template and add your API keys
cp .env.example .env
nano .env  # Add at least one API key
```

### 2. Launch All Services

```bash
docker compose up --build -d
```

This starts 6 services:

| Service | Port | Description |
|---------|------|-------------|
| **Frontend** | [localhost:3001](http://localhost:3001) | Next.js dashboard |
| **API** | [localhost:8080](http://localhost:8080) | FastAPI gateway |
| **Swagger Docs** | [localhost:8080/docs](http://localhost:8080/docs) | Interactive API docs |
| **Redis** | 6379 | Message broker |
| **PostgreSQL** | 5433 | Database |
| **Worker** | — | Celery background processor |

### 3. Use the Dashboard

Open [http://localhost:3001](http://localhost:3001), paste a YouTube URL, select your AI engine, and click **Generate Viral Clips**.

### 4. Or Use the CLI

```bash
# Inside the Docker container
docker compose exec api python main.py "https://www.youtube.com/watch?v=VIDEO_ID"

# With options
docker compose exec api python main.py "URL" --engine gemini --clips 5
```

---

## 📡 API Reference

### Health Check
```http
GET /health
```
```json
{ "status": "ok", "service": "autoclip-api" }
```

### Submit Clipping Job
```http
POST /api/v1/clip
Content-Type: application/json

{
  "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
  "preferred_engine": "auto",
  "num_clips": 5
}
```
```json
{
  "task_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "message": "Video added to processing queue",
  "status": "processing"
}
```

### Poll Task Status
```http
GET /api/v1/tasks/{task_id}
```
```json
{
  "task_id": "a1b2c3d4-...",
  "task_status": "SUCCESS",
  "result": {
    "status": "success",
    "clips_dir": "downloads/my-video/",
    "effects_plan_used": true,
    "seo_generated": true
  }
}
```

> 📖 Full interactive docs at [localhost:8080/docs](http://localhost:8080/docs) (Swagger UI)

---

## ⚙️ Configuration

### Environment Variables

```bash
# AI Engines (at least one required)
GEMINI_API_KEY=your_key        # Google Gemini 2.0 Flash
DEEPSEEK_API_KEY=your_key      # DeepSeek V3
GROQ_API_KEY=your_key          # Groq Llama 3.3 (also used for transcription)
ALIBABA_API_KEY=your_key       # Alibaba Qwen Plus

# Infrastructure (auto-configured in Docker)
REDIS_URL=redis://redis:6379/0
DATABASE_URL=postgresql://user:password@postgres:5432/autoclip
```

### Pipeline Settings (`config/settings.yaml`)

```yaml
pipeline:
  engine: auto          # auto | gemini | deepseek | groq | qwen
  max_clips: 7
  min_clip_duration: 30
  max_clip_duration: 90

effects:
  platforms:
    - tiktok
    - reels
    - facebook
```

---

## 🔬 The 6-Step Pipeline

```
Step 1  📥  Download video & extract audio (yt-dlp + FFmpeg)
   ↓
Step 2  📝  Transcribe audio (Groq Whisper API or local Whisper)
   ↓
Step 3  🧠  AI identifies viral moments from transcription
   ↓
Step 4  🎨  AI generates per-platform effects plan
   ↓
Step 5  ✂️  Clip, crop, face-track, subtitle, and render (FFmpeg + MediaPipe)
   ↓
Step 6  🔍  Generate SEO descriptions per clip per platform
```

Each step writes intermediate JSON files for debuggability and caching.

---

## 🧪 Testing

```bash
# Run unit tests
docker compose exec api pytest tests/ -v

# Type checking
docker compose exec api mypy --config-file mypy.ini .

# Lint
docker compose exec api flake8 --max-line-length 120
```

---

## 🛣️ Roadmap

- [x] Multi-engine AI analysis with fallback
- [x] MediaPipe face tracking
- [x] Hormozi-style ASS subtitles
- [x] Multi-platform export (TikTok, Reels, Facebook)
- [x] AI Effects Director
- [x] SEO description generation
- [x] FastAPI + Celery async pipeline
- [x] Next.js dashboard with real-time polling
- [ ] User authentication (NextAuth + Google OAuth)
- [ ] Stripe payment integration
- [ ] Auto-posting to TikTok, YouTube, Instagram
- [ ] Video preview in dashboard
- [ ] Webhook notifications
- [ ] Analytics & usage tracking

---

## 🤝 Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**Built with ❤️ by [amineautoformation-dev](https://github.com/amineautoformation-dev)**

</div>
