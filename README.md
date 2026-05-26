# HRP — Integrated HR and Project Management System

> **SWR391 · Software Development Project · FPT University · 2026**

HRP (Human Resource & Project Management System) is a web-based internal platform that digitizes and automates HR operations and project management workflows within an organization. The system replaces fragmented manual processes (email, Excel, paper forms) with a unified, role-controlled platform supporting **6 actor types** and **54 use cases** across 8 functional modules.

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
| **System name** | HRP – Integrated HR and Project Management System |
| **Course** | SWR391 – Software Development Project |
| **Team size** | 5 members |
| **Target users** | Employee, Team Leader, HR Manager, General Manager, Administrator, Internal Agent |
| **Platform** | Web application (Chrome, Firefox, Edge) |
| **Document version** | SRS v1.0 |

**Problem statement:** The organization currently manages HR and project operations through scattered, manual tools — paper files, Excel sheets, email threads, and disconnected fingerprint devices. This creates data inconsistency, no standardized approval flows, missing audit trails, and an absence of role-based access control (RBAC), leading to delays, errors, and security risks in critical HR operations.

**Solution:** HRP provides a single integrated platform where:
- HR processes (attendance, payroll, leave/OT requests, recruitment) are fully digitized with standardized approval flows.
- Project and task management are unified with team-level permissions and real-time progress tracking.
- An Internal AI Agent enables employees to execute commands via natural language within their permission scope.
- Role-based access control (RBAC) is enforced across all 6 actor types, with full audit trail for sensitive operations.

---

## Research-Based Learning (RBL) Focus

This project is built around deliberate, research-driven learning in three areas:

### 1. Algorithms
- **AI Agent Command Processing** — the Internal Agent authenticates the requesting user's identity and permission scope before executing any natural-language command, preventing unauthorized actions.
- **SLA-aware Approval Routing** — leave, OT, remote work, and shift-swap requests are routed to the correct approver based on the employee's assigned Team Leader or HR Manager, with configurable deadline tracking.
- **Cron Job – Auto Payroll Generation** — a background scheduler automatically triggers payroll calculation per a schedule configured by the General Manager, reducing manual computation errors.

### 2. System & Architecture
- **Role-Based Access Control (RBAC)** — a 6-tier permission hierarchy (Employee → Team Leader → HR Manager → General Manager → Administrator → Internal Agent) enforced at the API level. Each role inherits the permissions of lower roles where applicable.
- **Fingerprint Sync Service** — a background sync service bridges hardware fingerprint devices to the attendance module, removing the need for manual data entry.
- **Audit Trail** — all sensitive operations (payroll edits, leave approvals, login events) are logged for traceability and compliance.
- **Social Platform API Integration** — recruitment job postings are published to LinkedIn and other platforms via API credentials configured by the Administrator.
- **Fail-safe email notifications** — automated emails for login credentials, approval results, and interview schedules are decoupled from core operations; email disruption delays notifications but does not affect system data integrity.

### 3. Technology
| Layer | Choice | Why |
|---|---|---|
| **Frontend** | React + Vite + TypeScript + Tailwind CSS | Component-driven UI, strong typing for complex form schemas and role-conditional views |
| **Backend** | Node.js + Express | Non-blocking I/O suitable for concurrent approval flows and background sync jobs; flexible middleware for RBAC, logging, auth |
| **Database** | PostgreSQL (18 core tables) | ACID compliance critical for payroll and attendance records; relational model fits RBAC and approval flow foreign keys |
| **AI / Agent** | Internal Agent service | Natural-language command parsing with identity verification and permission-scoped execution |
| **Deployment** | Vercel (FE) + Render (BE) + Supabase (DB) | Cost-effective for a student project; Render avoids cold-start issues of EC2 free tier |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Clients                              │
│    Browser (Employee / Team Leader / HR Manager /           │
│             General Manager / Administrator)                │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTPS
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Frontend  (React + Vite + TypeScript)          │
│                    Deployed on Vercel                       │
└────────────────────────┬────────────────────────────────────┘
                         │ REST API / JWT
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend  (Express.js + Node.js)                │
│                    Deployed on Render                       │
│  ┌──────────────────┐   ┌───────────────────────────────┐  │
│  │  Auth Middleware  │   │  Internal Agent Service       │  │
│  │  RBAC Enforcer   │   │  (NL command parser + authz)  │  │
│  │  Approval Router │   │  Fingerprint Sync Service     │  │
│  │  Cron Scheduler  │   │  Email Notification Service   │  │
│  └────────┬─────────┘   └──────────────┬────────────────┘  │
└───────────┼──────────────────────────── ┼───────────────────┘
            │                             │
     ┌──────▼──────┐              ┌───────▼────────────┐
     │  PostgreSQL  │              │  External Services │
     │  (Supabase)  │              │  Social Platform   │
     │  18 tables   │              │  API (LinkedIn...) │
     └─────────────┘              │  Fingerprint HW    │
                                  └────────────────────┘
```

---

## Tech Stack

```
Frontend   React 18 · Vite · TypeScript · Tailwind CSS
Backend    Node.js · Express.js · JWT · Nodemailer
Database   PostgreSQL · Supabase (managed, 18 core tables)
Agent      Internal AI Agent (NL command execution)
Deploy     Vercel · Render · Supabase
```

---

## Key Features

**For Employees**
- View and update personal profile
- Clock in/out via fingerprint device (auto-synced)
- Submit leave, overtime, remote work, shift-swap, and special-regime requests
- View personal payslips
- Participate in assigned projects and create/update tasks
- Send natural-language commands to the Internal Agent (within permission scope)

**For Team Leaders**
- All Employee capabilities
- Approve/reject leave and OT requests from direct reports
- Manage team members and project assignments
- Track task progress within managed projects

**For HR Managers**
- Manage employee records (add, update, deactivate, assign roles)
- Configure attendance shifts and approval workflows
- Configure payroll components and salary templates
- Manage recruitment: post jobs (with Social Platform API), handle candidate profiles, schedule interviews, process hiring proposals
- Generate and export payroll reports

**For General Managers**
- Approve finalized payroll runs
- Manage cross-team projects
- Configure the auto-payroll cron job schedule
- Access full organization-wide reports and dashboards in real time

**For Administrators**
- Manage system security: role assignments, account lock/unlock, session control
- Monitor login activity and detect suspicious behavior
- Configure Social Platform API credentials for recruitment integration
- Perform password resets for any account

---

## Getting Started

### Prerequisites

- Node.js ≥ 18
- PostgreSQL (or a Supabase project)
- Fingerprint Sync Service endpoint (for attendance hardware integration)
- Social Platform API credentials (for recruitment posting — configured by Administrator)

### Installation

```bash
# Clone the repository
git clone https://github.com/<your-org>/HRP.git
cd HRP

# Install frontend dependencies
cd frontend
npm install

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
SMTP_HOST=...
SMTP_USER=...
SMTP_PASS=...
FINGERPRINT_SYNC_URL=https://...
SOCIAL_API_KEY=...
CRON_PAYROLL_SCHEDULE=...
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
npm run dev
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
HRP/
├── frontend/               # React Vite application
│   ├── src/
│   │   ├── components/     # Reusable UI components
│   │   ├── pages/          # Route-level pages per role (Employee, TL, HR, GM, Admin)
│   │   ├── hooks/          # Custom React hooks
│   │   ├── services/       # API client functions
│   │   └── types/          # TypeScript type definitions
│   └── vite.config.ts
│
├── backend/                # Express.js API server
│   ├── src/
│   │   ├── routes/         # API route handlers (hrp.authentication, hrp.employee, etc.)
│   │   ├── middleware/      # Auth, RBAC enforcement, rate limiting, error handling
│   │   ├── services/        # Business logic (payroll engine, attendance sync, agent, notifications)
│   │   ├── models/          # Database models / queries (18 core tables)
│   │   └── utils/           # Helpers (OTP, file upload, cron scheduler, etc.)
│   └── migrations/          # PostgreSQL migration scripts
│
├── docs/
│   ├── SRS_v1.0.docx        # Software Requirements Specification
│   ├── architecture.drawio  # System architecture diagram
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
| TBD | AI Agent Integration |
| TBD | Database / DevOps |
| TBD | QA / Documentation |

---

## Documents

| Document | Description |
|---|---|
| [`docs/SRS_v1.0.docx`](docs/SRS_v1.0.docx) | Software Requirements Specification — actors, 54 use cases, functional & non-functional requirements, DB design |
| [`docs/architecture.drawio`](docs/architecture.drawio) | System architecture diagram (draw.io) |
| [`docs/paper.md`](docs/paper.md) | Research paper — RBAC and AI Agent in integrated HR systems |

---

> **HRP** — *One platform for every HR and project operation — accurate, transparent, and role-controlled.*
