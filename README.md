<div align="center">

<img src="assets/logo.svg" alt="NarrowExtract" width="480"/>

<br/>

**Production-grade AI document ingestion, extraction & review system — fully offline, zero API costs.**

<br/>

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![Python](https://img.shields.io/badge/Python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-a9cc54?style=for-the-badge&logo=celery&logoColor=ddf4a4)
![Redis](https://img.shields.io/badge/Redis-DD0031?style=for-the-badge&logo=redis&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-black?style=for-the-badge&logo=ollama&logoColor=white)

<br/>

[Features](#-features) • [Architecture](#-architecture) • [Getting Started](#-getting-started) • [Configuration](#-configuration) • [Demo](#-demo)

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-spidysan-181717?style=flat-square&logo=github)](https://github.com/spidysan)

</div>

---

## What is NarrowExtract?

**NarrowExtract** eliminates manual data entry by automatically ingesting documents — invoices, receipts, ID cards, forms — and extracting structured data from them using local LLMs via [Ollama](https://ollama.com/). Everything runs **100% offline**, with no third-party API costs.

High-confidence extractions are auto-accepted. Ambiguous ones are routed to a human review queue. The result: faster processing, fewer errors, and full control over your data pipeline.

---

## ✨ Features

| Feature | Description |
|---|---|
| 📥 **Automated Ingestion** | Upload manually (single or batch) or auto-ingest email attachments via Gmail IMAP |
| 🤖 **AI-Powered Extraction** | Local LLMs (Mistral 7B, Qwen, etc.) extract key-value pairs from complex layouts |
| 🧩 **Custom Schemas** | Define per-document-type extraction fields using a JSON schema editor in the UI |
| 👁️ **Human-in-the-Loop** | Side-by-side review UI for flagged low-confidence extractions |
| 📊 **Analytics Dashboard** | Real-time metrics: volume trends, auto-acceptance rates, processing times |
| 📤 **Flexible Export** | Export to CSV, Excel, or JSON with date filters and export history log |
| ⚙️ **Extensible Architecture** | Horizontally scalable via FastAPI + Celery + PostgreSQL |

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     React Frontend                       │
│         Dashboard · Upload · Review · Export             │
└──────────────────────┬──────────────────────────────────┘
                       │ REST API (Axios)
┌──────────────────────▼──────────────────────────────────┐
│                   FastAPI Backend                        │
│                                                         │
│   ┌─────────────┐   ┌──────────────┐  ┌─────────────┐  │
│   │  PostgreSQL  │   │ Celery+Redis │  │   Ollama    │  │
│   │  SQLAlchemy  │   │  Async Jobs  │  │  Local LLM  │  │
│   └─────────────┘   └──────────────┘  └─────────────┘  │
│                             │                           │
│                   ┌─────────▼──────────┐               │
│                   │  OCR Pipeline       │               │
│                   │  (Tesseract + PDFs) │               │
│                   └────────────────────┘               │
└─────────────────────────────────────────────────────────┘
```

**Tech Stack:**

- **Frontend:** React + Vite, React Router, custom dark glassmorphism CSS
- **Backend:** FastAPI, SQLAlchemy, PostgreSQL
- **Async Processing:** Celery + Redis
- **AI:** Ollama (local LLM inference — Mistral 7B, Qwen, etc.)
- **OCR:** Tesseract + custom pipelines

---

## 🚀 Getting Started

### Prerequisites

- Node.js v18+
- Python 3.10+
- PostgreSQL
- Redis
- [Ollama](https://ollama.com/) installed and running

### 1. Pull an LLM

```bash
ollama pull mistral:7b
```

> Any Ollama-compatible model works. You can switch models later in Settings.

### 2. Database Setup

Create a PostgreSQL database:

```sql
CREATE DATABASE narrowextract;
```

### 3. Backend Setup

```bash
cd narrowextract-backend

# Create and activate virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
```

Edit `.env`:

```env
DATABASE_URL=postgresql://user:password@localhost:5432/narrowextract
REDIS_URL=redis://localhost:6379/0
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_TEXT_MODEL=mistral:7b
```

Start the backend services:

```bash
# API server
uvicorn main:app --reload --port 8000

# Celery worker (new terminal)
celery -A app.workers.celery_app worker --loglevel=info

# Celery beat scheduler — for automated email ingestion (new terminal)
celery -A app.workers.celery_app beat --loglevel=info
```

### 4. Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## ⚙️ Configuration

From the **Settings** page in the app you can:

- Switch the active Ollama model at runtime
- Configure Gmail IMAP credentials for automatic email attachment ingestion
- Define **Custom Extraction Schemas** — specify exactly which fields to extract per document type (e.g. `Invoice Number`, `Total Amount`, `Vendor Name`)

---

## 📊 How It Works

```
Document Upload
      │
      ▼
OCR / Text Extraction
      │
      ▼
LLM Extraction + Confidence Scoring
      │
   ┌──┴────────────────┐
   │                   │
High Confidence     Low Confidence
   │                   │
Auto-Accept        Review Queue
   │                   │
   └──────┬────────────┘
          │
     Export (CSV / JSON / Excel)
```

---

## 📁 Project Structure

```
narrowextract/
├── narrowextract-backend/
│   ├── main.py
│   ├── app/
│   │   ├── models/          # SQLAlchemy models
│   │   ├── routers/         # FastAPI route handlers
│   │   ├── workers/         # Celery tasks
│   │   └── services/        # LLM + OCR integration
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── pages/           # Dashboard, Upload, Review, Export
    │   ├── components/
    │   └── styles/
    └── package.json
```

---

## 🗺 Roadmap

- [ ] Multi-language OCR support
- [ ] Webhook integrations (Zapier, Make)
- [ ] Role-based access control
- [ ] Docker Compose one-command setup
- [ ] Active learning from human corrections

---

## 🎥 Demo

> 🎬 **Video walkthrough coming soon** — will cover document upload, AI extraction, review queue, and export flow.

<!-- Once ready, replace this block with:
[![Watch the demo](https://img.youtube.com/vi/YOUR_VIDEO_ID/maxresdefault.jpg)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)
-->

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📝 License

This project is licensed under the [MIT License](LICENSE).

---

<div align="center">
  <sub>Built with ❤️ using FastAPI, React, and Ollama</sub>
</div>
