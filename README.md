<div align="center">

<img src="https://img.shields.io/badge/StudyMind-AI%20Study%20Agent-7c3aed?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cGF0aCBkPSJNOS41IDJBOS41IDIuNSAwIDAgMSAxMiA0LjV2MTVhMi41IDIuNSAwIDAgMS00Ljk2LS40NiAyLjUgMi41IDAgMCAxLTIuOTYtMy4wOCAzIDMgMCAwIDEtLjM0LTUuNTggMi41IDIuNSAwIDAgMSAxLjMyLTQuMjQgMi41IDIuNSAwIDAgMSAxLjk4LTNAOS41IDJaIiBzdHJva2U9IndoaXRlIiBzdHJva2Utd2lkdGg9IjEuOCIgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIiBzdHJva2UtbGluZWpvaW49InJvdW5kIi8+PC9zdmc+" />

# StudyMind — AI Study Agent

**An intelligent study agent that teaches you, quizzes you, and adapts to how you learn.**

[![Live Demo](https://img.shields.io/badge/Live-Demo-success?style=flat-square)](https://anshumanbahekar.github.io/studymind-agent)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat-square)](LICENSE)
[![MongoDB](https://img.shields.io/badge/Partner-MongoDB%20Atlas-00ED64?style=flat-square&logo=mongodb)](https://www.mongodb.com/atlas)
[![Gemini](https://img.shields.io/badge/Powered%20by-Gemini%202.5-4285F4?style=flat-square&logo=google)](https://aistudio.google.com)
[![Python](https://img.shields.io/badge/Backend-Python%20Flask-3776AB?style=flat-square&logo=python)](https://flask.palletsprojects.com)

[**Live App**](https://anshumanbahekar.github.io/studymind-agent) · [**Demo Video**](https://www.youtube.com/watch?v=BAF5gNougII) · [**Architecture**](#architecture)

</div>

---

## What is StudyMind?

StudyMind is a multi-agent AI system that moves far beyond a chatbot. You give it a topic — it teaches the concept in three modes (explain, lesson, or Socratic), generates flashcards, quizzes you using active recall, evaluates your answers with Gemini, schedules future reviews with the SM-2 spaced repetition algorithm, detects your weak areas, and generates personalised remediation plans.

**Every agent action reads or writes MongoDB Atlas via MCP. The database is not cosmetic — it is the intelligence layer.**

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER (Browser)                       │
│              GitHub Pages — React/HTML Frontend             │
└──────────────────────────┬──────────────────────────────────┘
                           │ REST API
┌──────────────────────────▼──────────────────────────────────┐
│                   ORCHESTRATOR AGENT                        │
│              Flask · Intent Detection · Routing             │
└────┬──────────────┬─────────────┬──────────────┬────────────┘
     │              │             │              │
┌────▼────┐  ┌──────▼──────┐ ┌───▼────┐  ┌──────▼──────┐
│INGESTOR │  │   TEACHER   │ │ TUTOR  │  │  PROGRESS   │
│  AGENT  │  │    AGENT    │ │ AGENT  │  │    AGENT    │
│         │  │             │ │        │  │             │
│Gemini → │  │3 teach modes│ │SM-2 +  │  │Aggregation  │
│concepts │  │Explain      │ │Answer  │  │pipelines +  │
│+cards   │  │Lesson       │ │eval +  │  │Weak area    │
│→MongoDB │  │Socratic     │ │Cache   │  │detection    │
└────┬────┘  └──────┬──────┘ └───┬────┘  └──────┬──────┘
     │              │             │              │
     └──────────────┴─────────────┴──────────────┘
                           │
              ┌────────────▼────────────┐
              │    MongoDB Atlas MCP    │
              │                         │
              │  topics · concepts      │
              │  flashcards · sessions  │
              │  users · eval_cache     │
              └─────────────────────────┘
```

---

## MongoDB MCP Integration

This is the core of the submission. Every agent action goes through the MongoDB MCP layer. Here are the key tool calls:

| Tool | Agent | Purpose |
|---|---|---|
| `mongodb_insert_one` | Ingestor | Store new topic document |
| `mongodb_insert_many` | Ingestor | Bulk insert flashcards |
| `mongodb_find` | Tutor | Fetch due cards sorted by confidence |
| `mongodb_update_one` | Tutor | Write SM-2 scores after each answer |
| `mongodb_aggregate` | Progress | Compute mastery, weak areas, streak |
| `mongodb_find_one` | Tutor | Cache lookup for instant evaluation |
| `mongodb_insert_one` | Tutor | Store evaluation in `eval_cache` |

### Example — SM-2 update written to Atlas after every answer

```json
{
  "$set": {
    "confidence_score": 0.72,
    "ease_factor": 2.6,
    "interval_days": 6,
    "next_review_at": "2026-06-14T00:00:00Z"
  },
  "$inc": { "times_seen": 1 }
}
```

### Example — Weak area detection pipeline

```js
db.flashcards.aggregate([
  { $match: { user_id: uid, times_seen: { $gt: 0 } } },
  { $group: { _id: "$concept_id", avg_confidence: { $avg: "$confidence_score" } } },
  { $match: { avg_confidence: { $lt: 0.6 } } },
  { $sort: { avg_confidence: 1 } },
  { $limit: 5 }
])
```

---

## Features

| Feature | Description |
|---|---|
| **3 Teaching Modes** | Explain (analogy + example), Lesson (step-by-step), Socratic (guided discovery) |
| **SM-2 Spaced Repetition** | Industry-standard algorithm schedules each card individually |
| **Answer Caching** | Evaluations stored in MongoDB — same answer = instant response, no Gemini call |
| **Weak Area Detector** | MongoDB aggregation finds concepts below 60% confidence, Gemini writes remediation |
| **Knowledge Graph** | Interactive canvas visualising concept relationships and mastery |
| **AI Exam Generator** | Timed exam with MCQ + short answer, auto-scored |
| **Study Plan Generator** | Week-by-week learning roadmap from any topic |
| **Session Tracking** | Every quiz session recorded → real streak computation |
| **Cross-device Sync** | Auth via MongoDB — same data on any device |
| **API Key Rotation** | Up to 50 Gemini keys rotate automatically on quota limit |

---

## Tech Stack

| Layer | Technology | Cost |
|---|---|---|
| AI | Gemini 2.5 Flash (Google AI Studio) | Free |
| Database + MCP | MongoDB Atlas M0 | Free |
| Backend | Python + Flask (Replit) | Free |
| Frontend | HTML/CSS/JS (GitHub Pages) | Free |

**Total cost: $0**

---

## Collections

```
studymind/
├── users           — registered accounts (hashed passwords)
├── sessions        — auth session tokens
├── topics          — ingested study topics
├── concepts        — concept breakdown per topic
├── flashcards      — SM-2 cards with confidence scores
├── quiz_sessions   — completed sessions for streak tracking
└── eval_cache      — cached Gemini evaluations (speed + cost)
```

---

## Setup

```bash
git clone https://github.com/anshumanbahekar/studymind-agent
cd studymind-agent
pip install -r requirements.txt
```

Add to Replit Secrets:
```
GEMINI_API_KEY = your_key
MONGODB_URI    = mongodb+srv://...
```

```bash
python main.py
```

---

## API

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Health check |
| `POST` | `/auth/register` | Create account |
| `POST` | `/auth/login` | Sign in |
| `POST` | `/teach` | Teach a concept (3 modes) |
| `POST` | `/ingest` | Generate + store flashcards |
| `POST` | `/quiz/next` | Fetch next due card |
| `POST` | `/quiz/answer` | Evaluate + SM-2 update |
| `GET` | `/progress` | Mastery, weak areas, streak |
| `POST` | `/topic/cards` | All cards for a topic |
| `POST` | `/session/record` | Record quiz session |
| `POST` | `/chat` | Natural language orchestrator |

---

## Hackathon Track

**Partner:** MongoDB — track integration via Atlas MCP across all 4 agents  
**Challenge:** Education / Personal productivity  
**Built with:** Gemini 2.5 Flash · MongoDB Atlas · Flask · GitHub Pages

---

## License

Apache 2.0 — see [LICENSE](LICENSE)
