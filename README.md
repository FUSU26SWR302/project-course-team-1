# EduDesk — Academic Request Management System with AI Triage

> **SWP391 · Software Development Project · FPT University · 2026**

EduDesk is a web-based platform that allows FPT University students to submit and track academic requests online 24/7. The system integrates AI to automatically triage simple requests and assists academic advisors in processing complex cases faster — following a **Human-in-the-Loop (HITL)** design principle.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Research-Based Learning (RBL) Focus](#research-based-learning-rbl-focus)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Key Features](#key-features)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Team](#team)
- [Documents](#documents)

---

## Project Overview

| Field | Details |
|---|---|
| **System name** | EduDesk – Academic Request Management System with AI Triage |
| **Course** | SWP391 – Software Development Project |
| **Team size** | 5 members |
| **Target users** | Students, Academic Advisors, Administrators at FPT University |
| **Platform** | Web application (responsive, desktop & tablet) |
| **Document version** | RDS v1.0 |

**Problem statement:** FPT University students frequently need to submit academic requests (class transfer, semester deferral, grade appeal, student confirmation, etc.). The current process is slow, opaque, and creates heavy workload on advisors — especially at the start and end of each semester.

**Solution:** EduDesk applies a HITL AI model where:
- AI auto-processes simple, clear-cut requests within minutes.
- Complex or uncertain cases are escalated to human advisors with AI-suggested responses.
- Every AI decision is auditable and overridable — no black box.

---

## Research-Based Learning (RBL) Focus

This project is built around deliberate, research-driven learning in three areas:

### 1. Algorithms
- **AI Triage Engine** — configurable confidence threshold (default 0.80) determines whether an academic request is auto-approved, auto-rejected, or escalated to a human advisor.
- **RAG (Retrieval-Augmented Generation)** — the FAQ chatbot retrieves answers strictly from an admin-managed knowledge base of real university policies, preventing hallucination.
- **SLA Priority Scheduling** — requests in the advisor queue are sorted by remaining SLA time, with color-coded urgency indicators (green / yellow / red).

### 2. System & Architecture
- **Human-in-the-Loop (HITL)** — the architectural pattern that sits at the center of every design decision. AI never has the final word; students can appeal within 3 days, and advisors can override at any time.
- **Fail-safe by design** — API errors always escalate to a human; the system never auto-rejects a request due to a technical failure.
- **Audit trail** — every AI action is recorded in `AI_LOG` (prompt, output, confidence score, override reason), satisfying SWP391's Responsible AI requirements.
- **Role-based access control** — three-tier permission system (Student / Advisor / Admin) enforced at the API level with JWT authentication.
- **Dynamic form schema** — request form fields are driven by admin-configured schemas per request type, making the system extensible without code changes.

### 3. Technology
| Layer | Choice | Why |
|---|---|---|
| **Frontend** | React + Vite + TypeScript + Tailwind CSS | Rich UI library ecosystem (`react-table`, `react-big-calendar`), leaner than Next.js for an internal admin portal, faster builds with Bun |
| **Backend** | Node.js + Express | Non-blocking I/O handles high-concurrency peaks (course registration periods); flexible middleware for rate limiting, logging, custom auth |
| **Database** | PostgreSQL via Supabase | ACID compliance critical for academic records; serverless, easy setup |
| **AI / Automation** | Openclaw (self-hosted) + N8N | Full control over AI pipeline; self-hosting avoids EC2 free-tier latency issues (verified by testing) |
| **Deployment** | Vercel (FE) + Render (BE) + Supabase (DB) | Cost-effective for a student project; Render avoids the cold-start and speed issues of EC2 free tier |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                              │
│          Browser (Student / Advisor / Admin)                │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Frontend  (React Vite + TypeScript)            │
│                    Deployed on Vercel                       │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API / JWT
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend  (Express.js + Node.js)                │
│                    Deployed on Render                       │
│  ┌──────────────────┐   ┌───────────────────────────────┐  │
│  │  Request Router  │   │  AI Triage Service            │  │
│  │  Auth Middleware │   │  (confidence threshold logic) │  │
│  │  SLA Scheduler   │   │  FAQ Chatbot (RAG)            │  │
│  └────────┬─────────┘   └──────────────┬────────────────┘  │
└───────────┼──────────────────────────── ┼───────────────────┘
            │                             │
     ┌──────▼──────┐              ┌───────▼────────┐
     │  PostgreSQL  │              │   Openclaw AI  │
     │  (Supabase)  │              │  (self-hosted) │
     └─────────────┘              └────────────────┘
```

---

## Tech Stack

```
Frontend   React 18 · Vite · TypeScript · Tailwind CSS · Bun
Backend    Node.js · Express.js · JWT · Nodemailer (OTP)
Database   PostgreSQL · Supabase (managed)
AI         Openclaw (self-hosted) · N8N (workflow automation)
Deploy     Vercel · Render · Supabase
```

---

## Key Features

**For Students**
- Submit academic requests via dynamic, type-aware forms
- Attach supporting documents (PDF / JPG, max 5 MB × 3 files)
- Track request status on a visual timeline: Submitted → AI Processing → Result / Escalated → Closed
- Chat with the 24/7 AI FAQ bot — answers cited from official policy documents
- Appeal any AI decision within 3 days

**For Academic Advisors**
- SLA-prioritized request queue with color-coded urgency
- Full request detail view: content, attachments, AI analysis, suggested response
- Approve / reject / edit AI suggestions before sending to student
- Override any auto-processed AI decision (with mandatory reason)

**For Administrators**
- Configure request types, form schemas, SLA deadlines, and AI confidence thresholds
- Manage users (create, import via CSV, assign roles, lock/unlock)
- Manage chatbot knowledge base (upload / update / delete policy documents)
- Dashboard: AI self-processing rate, SLA compliance, advisor workload, override frequency
- Export reports to CSV

---

## Getting Started

### Prerequisites

- Node.js ≥ 18 / Bun ≥ 1.0
- PostgreSQL (or a Supabase project)
- Openclaw instance (self-hosted) or compatible AI API endpoint

### Installation

```bash
# Clone the repository
git clone https://github.com/<your-org>/edudesk.git
cd edudesk

# Install frontend dependencies
cd frontend
bun install

# Install backend dependencies
cd ../backend
npm install
```

### Environment Variables

Create `.env` files in `frontend/` and `backend/` based on the provided `.env.example` files.

**backend/.env**
```
DATABASE_URL=postgresql://...
JWT_SECRET=...
AI_API_URL=https://your-openclaw-instance/...
AI_API_KEY=...
SMTP_HOST=...
SMTP_USER=...
SMTP_PASS=...
```

**frontend/.env**
```
VITE_API_BASE_URL=http://localhost:3000
```

### Running Locally

```bash
# Start backend (from /backend)
npm run dev

# Start frontend (from /frontend)
bun run dev
```

The frontend will be available at `http://localhost:5173` and the backend at `http://localhost:3000`.

### Database Setup

```bash
# Run migrations (from /backend)
npm run migrate

# Seed sample data
npm run seed
```

---

## Project Structure

```
edudesk/
├── frontend/               # React Vite application
│   ├── src/
│   │   ├── components/     # Reusable UI components
│   │   ├── pages/          # Route-level pages (Student, Advisor, Admin)
│   │   ├── hooks/          # Custom React hooks
│   │   ├── services/       # API client functions
│   │   └── types/          # TypeScript type definitions
│   └── vite.config.ts
│
├── backend/                # Express.js API server
│   ├── src/
│   │   ├── routes/         # API route handlers
│   │   ├── middleware/      # Auth, rate limiting, error handling
│   │   ├── services/        # Business logic (AI triage, SLA, notifications)
│   │   ├── models/          # Database models / queries
│   │   └── utils/           # Helpers (OTP, file upload, etc.)
│   └── migrations/          # PostgreSQL migration scripts
│
├── docs/
│   ├── RDS_v1.0.docx        # Requirements Description Specification
│   ├── architecture.drawio  # System architecture diagram
│   ├── SRS.md               # Software Requirements Specification
│   └── paper.md             # Research paper
│
└── README.md
```

---

## Team

| Member | Role |
|---|---|
| TBD | Team Lead / Backend |
| TBD | Frontend |
| TBD | AI Integration |
| TBD | Database / DevOps |
| TBD | QA / Documentation |

---

## Documents

| Document | Description |
|---|---|
| [`docs/RDS_v1.0.docx`](docs/RDS_v1.0.docx) | Requirements Description Specification — actors, use cases, functional scope |
| [`docs/SRS.md`](docs/SRS.md) | Software Requirements Specification — detailed functional & non-functional requirements |
| [`docs/architecture.drawio`](docs/architecture.drawio) | System architecture diagram (draw.io) |
| [`docs/paper.md`](docs/paper.md) | Research paper — HITL AI in academic request management |

---

> **EduDesk** — *Because every student's request deserves a fast, fair, and transparent answer.*