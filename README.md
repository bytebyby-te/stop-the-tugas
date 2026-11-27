# 📚 Stop The Tugas

**Stop The Tugas** ([`constellation-a2/stop-the-tugas`](https://github.com/constellation-a2/stop-the-tugas)) is a collaborative academic management web application built with Python to help students track assignment deadlines, receive timely class and deadline reminders, view calendar agendas per date, and organize lecture materials and notes in one place.  
---

## 🎯 MVP Scope

### In Scope (v1)
- **Reminders & Notifications (Team B):**
  - Browser **Web Push Notifications** (via Service Worker, Push API, and VAPID keys) delivered even when the browser tab/window is closed, after the user enables notifications.
  - Automated assignment reminders at **H-7**, **H-3**, and **H-1** before deadlines in **`WIB (Asia/Jakarta)`**; assignments added with fewer days remaining (e.g., H-2) only schedule remaining intervals (e.g., H-1), and past-deadline items trigger no new reminders.
  - Class schedule reminders at **H-1** (e.g., `19:00 WIB` the day before) with course name, time, and room, plus **H-30 minutes** before class start time.
  - Automatic cancellation or recalculation of pending reminders when Team C deletes or modifies an agenda, plus clear empty-date states (e.g., *"Tidak ada tugas atau jadwal kuliah pada tanggal ini"*).
  - Duplicate-delivery prevention during scheduled cron runs via `claimed_at` and PostgreSQL row locking (`FOR UPDATE SKIP LOCKED`) without a separate login system.
- **Calendar & Schedules (Team C):**
  - Open **Create** and **Delete** workflows for assignments (`TUGAS`) and class schedules (`KULIAH`), including title, date & time (start, optional end/deadline), optional description/notes, type, and classroom location (`class_loc`) for classes.
  - Confirmation prompt before deleting an agenda item so accidental deletions are avoided and pending reminders are cancelled cleanly.
  - Monthly calendar view highlighting dates with agendas and daily agenda listings in **`WIB`**.
- **Materials & Notes Repository (Team D):**
  - **CRD** operations (**Create, Read, Delete** without **Update**) for study materials and notes—no editing/updating of files or metadata.
  - Strict validation accepting **`.pdf`** files only (`application/pdf` & PDF header check) up to **10 MB** per file, returning clear errors otherwise.
  - Required metadata: **Course Label** (dropdown shared across Teams B, C, and D), **Title**, and **Creator** (populated via input / browser `localStorage` without login), with preview/download and delete confirmation.

### Out of Scope (No Login & Post-MVP)
- **Authentication / Login System:** Explicitly out of scope for MVP (no user accounts or login flow).
- Editing/updating existing schedules or uploaded materials/metadata (CRD without Update).
- Complex role-based access control (RBAC), multi-level approval workflows, or advanced analytics dashboards.

---

## 🏗️ Tech Stack & Architecture

- **Backend & Web Framework:** Python (Flask)
- **Frontend & Client Logic:** HTML5, CSS3, JavaScript (including Service Worker `sw.js` for Web Push), & PyScript
- **Database:** PostgreSQL hosted on [Neon](https://neon.tech) (modeled with Oracle Data Modeler)
- **Scheduling & Push:** Scheduled cron jobs (`WIB / Asia/Jakarta`), Web Push API, VAPID, & Service Worker
- **File Storage:** Persistent object storage (e.g., Vercel Blob / Supabase Storage) for PDFs on serverless deployment (avoiding ephemeral local disk on Vercel)
- **Deployment:** [Vercel](https://vercel.com/docs/deployments)

---

## 🗄️ Database Design (ERD & Physical Model)

Database artifacts created in **Oracle Data Modeler** and adapted for **PostgreSQL (Neon)** are stored in the `database/` directory:

- **ERD & Oracle Data Modeler Project:** `database/erd/stop_the_tugas.dmd`, the `database/erd/stop_the_tugas/` project folder, and exported diagram image(s) (e.g., `database/erd/erd-diagram.png`).
- **Physical Schema (DDL):** `database/physical/schema.sql` (and optional seed data in `database/physical/seed.sql`).
- **Database Notes:** See `database/README.md` for table definitions, `TIMESTAMPTZ` / `WIB` handling, `ON DELETE CASCADE` reminder cleanup, and `claimed_at` cron locking.

---

## 🎨 UI/UX & Flowchart/Algorithm Documentation

### UI/UX Team
- **Design Source:** Add your Figma / design file link and key screens under `design/` (e.g., `design/design-system.md` and `design/screens/`).
- **Core MVP States to Document:** Monthly calendar markers, daily schedule view & empty-date message, agenda/material delete confirmation prompts, PDF `.pdf` / `<= 10 MB` validation feedback, and Web Push opt-in prompt.

### Flowchart & Algorithm Team
- **Detailed Specs & Diagrams:** Store step-by-step algorithms and diagrams under `docs/` (e.g., `docs/algorithms-and-flowcharts.md` and `docs/diagrams/`).
- **Core Flows Covered:** `WIB` reminder calculation (`H-7`, `H-3`, `H-1`, `H-1 19:00 WIB`, `H-30M`), agenda update/delete reminder sync, `FOR UPDATE SKIP LOCKED` + `claimed_at` cron dispatch, calendar queries, and CRD PDF validation.

---

## 📌 Project Management (GitHub Project & Milestones)

We coordinate work across **Team B**, **Team C**, **Team D**, **UI/UX**, and **Flowchart/Algorithm** using **GitHub Projects** and **GitHub Milestones**:

- **GitHub Project Board:** Track tasks across columns (`Backlog`, `Ready`, `In Progress`, `In Review / Testing`, `Done`) with fields/labels for team ownership (`team-b`, `team-c`, `team-d`, `ui-ux`, `algo-flowchart`, `db`) and priority (`P0`, `P1`, `P2`).
- **Milestones:** Group issues by delivery checkpoint (e.g., *M1: ERD, Schema & Specs*, *M2: Core CRUD/CRD & UI*, *M3: Web Push, Cron & Integration*, *M4: MVP QA & Demo*) and mark `blocked` issues early during check-ins.

---

## 📁 Project Structure

- [`.github/`](.github/)
  - [`workflows/`](.github/workflows/)
    - [`ci.yml`](.github/workflows/ci.yml) — GitHub Actions CI running lint and `pytest` on PRs to `dev`
- [`api/`](api/) — Serverless entrypoint for Vercel deployment
- [`app/`](app/)
  - [`__init__.py`](app/__init__.py) — Flask app initialization and configuration
  - [`models/`](app/models/) — PostgreSQL (Neon) data access / queries
  - [`routes/`](app/routes/)
    - [`reminder.py`](app/routes/reminder.py) — Push subscription & reminder/cron endpoints (Team B)
    - [`calendar.py`](app/routes/calendar.py) — Calendar & daily/monthly schedule endpoints (Team C)
    - [`materials.py`](app/routes/materials.py) — CRD materials & PDF validation endpoints (Team D)
  - [`services/`](app/services/) — Business logic (WIB countdowns, cron locking, validation)
  - [`static/`](app/static/)
    - [`css/`](app/static/css/) — Stylesheets
    - [`js/`](app/static/js/) — Client scripts & Service Worker (`sw.js`)
    - [`uploads/`](app/static/uploads/) — Local upload storage for PDF files (local dev only)
  - [`templates/`](app/templates/) — HTML templates & PyScript integration
- [`database/`](database/)
  - [`erd/`](database/erd/) — Oracle Data Modeler (`.dmd`), project folder, & ERD exports
  - [`physical/`](database/physical/) — Physical SQL scripts (`schema.sql`, `seed.sql`)
- [`design/`](design/) — UI/UX notes, design tokens, & screen references
- [`docs/`](docs/) — Flowcharts, algorithms, & technical documentation
- [`migrations/`](migrations/) — Database migration / SQL initialization scripts
- [`tests/`](tests/)
  - [`conftest.py`](tests/conftest.py) — Shared `pytest` fixtures (Flask client, PDFs, WIB time helpers)
  - [`unit/`](tests/unit/) — Unit tests for WIB reminders, calendar rules, and PDF validation
  - [`integration/`](tests/integration/) — Integration tests for cron locking, agenda routes, and PDF CRD routes
  - [`fixtures/`](tests/fixtures/) — Sample test files and PDF fixtures
- [`AGENTS.md`](AGENTS.md) — AI coding agent guidelines and Python best practices
- [`.env.example`](.env.example) — Example environment variables (VAPID, DB, Cron secret)
- [`requirements.txt`](requirements.txt) — Python dependencies
- [`vercel.json`](vercel.json) — Vercel build, routing, & cron configuration
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — Contribution, branching, & team workflow guidelines
- [`LICENSE`](LICENSE) — Usage & non-commercial license terms
- [`README.md`](README.md) — Project documentation

---

## 🤝 Contributing

We welcome contributions from all team members and collaborators. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening an issue or pull request for details on:
* Branch naming conventions, Git workflow, and GitHub Project / Milestone updates
* Module ownership and boundaries across **UI/UX**, **Database Engineering**, **Flowchart/Algorithm**,  **Team B**, **Team C**, and **Team D**
* Testing checklists, WIB (Asia/Jakarta) time handling, and security/privacy rules (never commit .env, VAPID private keys, credentials, or copyrighted files)
---

## 📄 License

This project is maintained by **`constellation-a2`** for academic and educational purposes.
See [`LICENSE`](LICENSE) for full details.