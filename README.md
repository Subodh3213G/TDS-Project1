# Automated App Builder, Deployer & Evaluator

## Overview
This project implements an **automated platform** that receives structured JSON task requests, generates apps using **LLM assistance**, deploys them to **GitHub Pages**, and reports deployment metadata to a central **evaluation server**.  
It supports **multiple rounds (build & revise)** per task — allowing iterative updates based on instructor feedback.

---

## Core Features

### Build Phase
- Receives JSON requests containing student info, task metadata, and app briefs.
- Verifies a secret for authentication.
- Calls **Gemini LLM API** to generate app files (`index.html`, `README.md`, `LICENSE`).
- Initializes or updates a GitHub repository:
  - Pushes generated files.
  - Enables **GitHub Pages** hosting.
- Notifies an external evaluation API with deployment details.

### Revise Phase
- Accepts a second JSON request for updates.
- Uses **surgical LLM edits** to modify existing code minimally.
- Commits and redeploys the app automatically.

###  Instructor Evaluation
- Evaluates generated apps via static, dynamic, and LLM-based checks using Playwright and rule-based logic.

---

## Tech Stack

| Component | Purpose |
|------------|----------|
| **FastAPI** | API framework for request handling |
| **httpx** | Async HTTP client |
| **GitPython** | Automates Git repo creation and commits |
| **Gemini API** | LLM used for app generation |
| **Uvicorn** | ASGI server |
| **Docker** | Containerization |
| **GitHub Pages** | Static hosting for generated apps |

---

## Project Structure
```
├── main.py              # Core FastAPI application and orchestration logic
├── requirements.txt     # Python dependencies
├── Dockerfile           # Container build configuration
├── logs/                # Runtime logs
└── generated_tasks/     # Folder where generated apps are stored
```

---

## Environment Variables

| Variable | Description |
|-----------|-------------|
| `GEMINI_API_KEY` | API key for Gemini model |
| `GITHUB_TOKEN` | GitHub personal access token (repo + pages scope) |
| `GITHUB_USERNAME` | GitHub username |
| `STUDENT_SECRET` | Shared secret to verify task requests |
| `LOG_FILE_PATH` | Optional log file path |
| `MAX_CONCURRENT_TASKS` | Optional concurrency control |
| `KEEP_ALIVE_INTERVAL_SECONDS` | Optional heartbeat interval |

Create a `.env` file:
```bash
GEMINI_API_KEY=your_api_key
GITHUB_TOKEN=your_token
GITHUB_USERNAME=your_username
STUDENT_SECRET=your_secret
```

---

## Docker Setup

### Build and Run
```bash
docker build -t auto-app-builder .
docker run -d -p 8000:8000 --env-file .env auto-app-builder
```

### View Logs
```bash
docker logs -f <container_id>
```

---

## API Endpoints

| Endpoint | Method | Description |
|-----------|--------|-------------|
| `/` | GET | Root message |
| `/ready` | POST | Accepts and processes task requests |
| `/status` | GET | Returns current task status |
| `/health` | GET | Health check |
| `/logs` | GET | View application logs |
| `/debug-secret` | GET | Returns configured secret (for debugging only) |

### Example Task Request
```bash
curl -X POST https://example.com/ready \
  -H "Content-Type: application/json" \
  -d '{
    "email": "student@example.com",
    "secret": "YOUR_SHARED_SECRET",
    "task": "captcha-solver-demo",
    "round": 1,
    "nonce": "abc123",
    "brief": "Build a captcha solver for ?url=...",
    "evaluation_url": "https://example.com/notify",
    "attachments": []
  }'
```

---

## Local Development

### Install Dependencies
```bash
pip install -r requirements.txt
```

### Run the Server
```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

### Test
```bash
curl http://localhost:8000/health
```

---

## License
This project is licensed under the [MIT License](./LICENSE).

---

## Author
**LexiSolve Student Automation Framework**  
Developed for automated LLM-driven app generation, deployment, and evaluation pipelines.
