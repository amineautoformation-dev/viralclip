# Contributing to ViralClip

Thank you for your interest in contributing to ViralClip! This guide will help you get started.

## 🏗️ Development Setup

### Prerequisites

- Python 3.10+
- Node.js 20+
- Docker & Docker Compose v2+
- At least one AI API key (Gemini, DeepSeek, Groq, or Alibaba)

### Local Setup

```bash
# Clone the repo
git clone https://github.com/amineautoformation-dev/viralclip.git
cd viralclip

# Copy environment template
cp .env.example .env
# Edit .env and add your API keys

# Start all services
docker compose up --build -d

# Install frontend dependencies (for local dev)
cd frontend && npm install && cd ..
```

### Running Tests

```bash
docker compose exec api pytest tests/ -v
docker compose exec api mypy --config-file mypy.ini .
docker compose exec api flake8 --max-line-length 120
```

## 📋 Contribution Workflow

1. **Fork** the repository
2. Create a **feature branch**: `git checkout -b feature/my-feature`
3. **Commit** your changes: `git commit -m "feat: add my feature"`
4. **Push** to your fork: `git push origin feature/my-feature`
5. Open a **Pull Request**

## 🏷️ Commit Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Usage |
|--------|-------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `docs:` | Documentation |
| `refactor:` | Code refactoring |
| `test:` | Adding tests |
| `chore:` | Maintenance |

## 📁 Where to Contribute

| Area | Files | Good for |
|------|-------|----------|
| **New AI Engine** | `engines/` | Add support for Claude, GPT-4, etc. |
| **Video Effects** | `effects_*.py` | New transitions, filters, styles |
| **Frontend** | `frontend/src/` | UI improvements, new pages |
| **API Endpoints** | `api/main.py` | New routes, middleware |
| **Tests** | `tests/` | Unit & integration tests |
| **Documentation** | `*.md` | Guides, examples, translations |

## 🐛 Reporting Bugs

Open an issue with:
- Steps to reproduce
- Expected vs actual behavior
- Environment details (OS, Docker version, API keys used)

## 📄 License

By contributing, you agree that your contributions will be licensed under the MIT License.
