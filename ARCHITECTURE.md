# 🏛️ Architecture — ViralClip

> Detailed technical architecture of the ViralClip platform.

---

## System Overview

ViralClip is built as a **distributed, event-driven microservices architecture** designed for horizontal scalability.

```mermaid
graph TB
    subgraph Client Layer
        A["🌐 Next.js Frontend<br/>localhost:3001"]
        B["📱 CLI<br/>main.py"]
    end

    subgraph API Layer
        C["⚡ FastAPI Gateway<br/>localhost:8080"]
    end

    subgraph Processing Layer
        D["🔄 Celery Worker"]
        E["📦 Redis<br/>Message Broker"]
    end

    subgraph Storage Layer
        F["🗄️ PostgreSQL<br/>User & Job Data"]
        G["📁 File System<br/>Videos & Clips"]
    end

    A -->|REST API| C
    B -->|Direct Call| D
    C -->|Enqueue Task| E
    E -->|Dequeue| D
    C -->|Read/Write| F
    D -->|Read/Write| F
    D -->|Save Media| G
```

---

## Processing Pipeline

The core pipeline runs as a **6-stage sequential workflow** inside a Celery worker:

```mermaid
flowchart LR
    A["📥 Download<br/><i>yt-dlp</i>"] --> B["📝 Transcribe<br/><i>Whisper</i>"]
    B --> C["🧠 Analyze<br/><i>Multi-AI</i>"]
    C --> D["🎨 Effects Plan<br/><i>AI Director</i>"]
    D --> E["✂️ Clip & Render<br/><i>FFmpeg + MediaPipe</i>"]
    E --> F["🔍 SEO Generate<br/><i>Multi-AI</i>"]

    style A fill:#1e3a5f,stroke:#4fc3f7,color:#fff
    style B fill:#1b5e20,stroke:#66bb6a,color:#fff
    style C fill:#4a148c,stroke:#ce93d8,color:#fff
    style D fill:#e65100,stroke:#ffb74d,color:#fff
    style E fill:#b71c1c,stroke:#ef9a9a,color:#fff
    style F fill:#006064,stroke:#4dd0e1,color:#fff
```

### Stage Details

| Stage | Module | Input | Output | AI Engine |
|-------|--------|-------|--------|-----------|
| 1. Download | `downloader.py` | YouTube URL | `.mp4` + `.mp3` | — |
| 2. Transcribe | `transcriber.py` | `.mp3` | `transcription.json` | Groq Whisper / Local Whisper |
| 3. Analyze | `analyzer.py` | `transcription.json` | `viral_moments.json` | Gemini → DeepSeek → Groq → Qwen |
| 4. Effects | `effects_director.py` | `viral_moments.json` | `effects.json` | Same fallback chain |
| 5. Clip | `clipper.py` | `.mp4` + all JSONs | Platform clips | — (FFmpeg + MediaPipe) |
| 6. SEO | `seo_generator.py` | All JSONs | `seo.json` | Same fallback chain |

---

## AI Engine Fallback Strategy

The engine manager implements a **strategy pattern with automatic fallback**:

```mermaid
flowchart TD
    A{"API Key Available?"}
    A -->|GEMINI_API_KEY| B["Gemini 2.0 Flash"]
    A -->|DEEPSEEK_API_KEY| C["DeepSeek V3"]
    A -->|GROQ_API_KEY| D["Groq Llama 3.3"]
    A -->|ALIBABA_API_KEY| E["Alibaba Qwen Plus"]

    B -->|"❌ Rate Limit / Error"| C
    C -->|"❌ Rate Limit / Error"| D
    D -->|"❌ Rate Limit / Error"| E
    E -->|"❌ All Failed"| F["🚫 Pipeline Error"]

    B -->|"✅ Success"| G["📄 JSON Response"]
    C -->|"✅ Success"| G
    D -->|"✅ Success"| G
    E -->|"✅ Success"| G

    style B fill:#1a73e8,stroke:#4285f4,color:#fff
    style C fill:#0d47a1,stroke:#2196f3,color:#fff
    style D fill:#e65100,stroke:#ff9800,color:#fff
    style E fill:#6a1b9a,stroke:#ab47bc,color:#fff
```

### Engine Comparison

| Engine | Speed | Quality | Cost | Best For |
|--------|:-----:|:-------:|:----:|----------|
| Gemini 2.0 Flash | ⚡⚡⚡ | ★★★★★ | Free tier | Primary analysis |
| DeepSeek V3 | ⚡⚡ | ★★★★☆ | $0.14/1M tokens | Complex analysis |
| Groq Llama 3.3 | ⚡⚡⚡⚡ | ★★★☆☆ | Free tier | Fast iteration |
| Alibaba Qwen Plus | ⚡⚡ | ★★★★☆ | $0.10/1M tokens | Fallback |

---

## Docker Services Topology

```mermaid
graph TB
    subgraph docker-compose.yml
        FE["🖥️ frontend<br/>node:20-alpine<br/>:3001→3000"]
        API["⚡ api<br/>Python 3.10<br/>:8080→8000"]
        WK["⚙️ worker<br/>Python 3.10<br/>Celery"]
        RD["📦 redis<br/>redis:7-alpine<br/>:6379"]
        PG["🗄️ postgres<br/>postgres:15-alpine<br/>:5433→5432"]
    end

    FE -->|"depends_on"| API
    API -->|"depends_on"| RD
    API -->|"depends_on"| PG
    WK -->|"depends_on"| RD
    WK -->|"depends_on"| PG

    FE -.->|"NEXT_PUBLIC_API_URL"| API
    API -.->|"REDIS_URL"| RD
    API -.->|"DATABASE_URL"| PG
    WK -.->|"REDIS_URL"| RD

    style FE fill:#000,stroke:#fff,color:#fff
    style API fill:#009688,stroke:#4db6ac,color:#fff
    style WK fill:#37814a,stroke:#66bb6a,color:#fff
    style RD fill:#d32f2f,stroke:#ef5350,color:#fff
    style PG fill:#1565c0,stroke:#42a5f5,color:#fff
```

---

## Data Flow

```mermaid
sequenceDiagram
    participant U as 👤 User
    participant F as 🌐 Frontend
    participant A as ⚡ API
    participant R as 📦 Redis
    participant W as ⚙️ Worker
    participant FS as 📁 FileSystem

    U->>F: Paste YouTube URL
    F->>A: POST /api/v1/clip
    A->>R: Enqueue task
    A-->>F: { task_id: "abc123" }

    F->>A: GET /api/v1/tasks/abc123
    Note over F,A: Poll every 3s

    R->>W: Dequeue task
    W->>FS: Download video (yt-dlp)
    W->>W: Transcribe (Whisper)
    W->>W: Analyze (AI Engine)
    W->>W: Effects Planning
    W->>FS: Clip & Render (FFmpeg)
    W->>W: Generate SEO
    W->>R: Task SUCCESS + result

    A-->>F: { status: "SUCCESS", result: {...} }
    F-->>U: ✅ Clips ready!
```

---

## Face Tracking Pipeline

```mermaid
flowchart LR
    A["🎥 Video Frame"] --> B["👤 MediaPipe<br/>Face Detection"]
    B --> C{"Face Found?"}
    C -->|Yes| D["📐 Calculate<br/>Bounding Box"]
    C -->|No| E["🎯 Use Previous<br/>or Center Crop"]
    D --> F["✂️ 9:16 Vertical<br/>Crop Window"]
    E --> F
    F --> G["🎬 FFmpeg<br/>Crop & Render"]

    style B fill:#4caf50,stroke:#81c784,color:#fff
    style F fill:#ff9800,stroke:#ffb74d,color:#fff
```

---

## Tech Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | Next.js 15, Tailwind CSS, shadcn/ui | Dashboard & Landing Page |
| **API** | FastAPI, Pydantic, Uvicorn | REST Gateway |
| **Queue** | Celery 5.4, Redis 7 | Async Task Processing |
| **Database** | PostgreSQL 15, SQLAlchemy, Alembic | Persistence |
| **AI** | Gemini, DeepSeek, Groq, Qwen | Analysis & Generation |
| **Video** | FFmpeg, yt-dlp | Download, Clip, Render |
| **ML** | MediaPipe, OpenAI Whisper | Face Tracking, Transcription |
| **Quality** | mypy, flake8, pytest | Type Safety & Testing |
| **Deploy** | Docker Compose | Orchestration |

---

## Security Considerations

- API keys stored in `.env` (never committed — protected by `.gitignore`)
- CORS restricted to `localhost:3000` and `localhost:3001`
- Pydantic validation on all API inputs
- No direct database exposure (only via API)
- Docker network isolation between services
